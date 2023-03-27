# 在 Ruby on Rails 中对表单对象使用 Dry-rb

> 原文：<https://itnext.io/using-dry-rb-for-form-objects-in-ruby-on-rails-f965e0c0e86a?source=collection_archive---------4----------------------->

Form 对象允许你将验证从 ActiveRecord 模型转移到单独的类，这允许你保持你的模型精简，这对于大型项目尤其重要。

我以前用 [virtus gem](https://github.com/solnic/virtus) 来建造形态物体，但是现在它被弃用了，为此我选择了 [dry-rb](https://dry-rb.org/) 。还有很多其他的宝石，比如浅属性，快属性，属性，也许你会选择它们。

**给宝石文件添加宝石**

```
gem 'dry-struct', '~> 1.2.0'
gem 'dry-types', '~> 1.2.2'
gem 'dry-validation', '~> 1.4.0'
```

我使用 dry-struct 作为表单对象的基类，dry-types 用于定义变量类型，dry-validation 用于验证对象。

**安装它**

```
bundle install
```

**建筑形体对象**

在我的应用程序中，我有一个带有名称和区域字符串属性的世界类，所以这个类的基本表单对象将是这样的。

```
class WorldForm < Dry::Struct
  module Types
    include Dry::Types(default: :nominal)
  end attribute :id, Types::Integer.optional
  attribute :name, Types::String
  attribute :zone, Types::Integer
end
```

在这里，我们为世界形态对象定义类并设置属性。在 [dry-types](https://dry-rb.org/gems/dry-types/1.2/) 中有许多属性选项可供您使用。

**添加验证**

为了在表单对象中使用验证，我添加了带有已定义模式和自定义验证规则的契约类。所以这里必须给出**名称**和**区域**字符串参数。在自定义验证规则中，我们首先传递规则中使用的属性，然后可以检查它们，所以我检查了具有这种参数的世界不存在(values[:id])。零？检查它是新记录还是已有记录)。

```
class WorldContract < Dry::Validation::Contract
  schema do
    optional(:id)
    required(:name).filled(:string)
    required(:zone).filled(:string)
  end rule(:id, :name, :zone) do
    worlds = values[:id].nil? ? World.all : World.where.not(id: values[:id]) key.failure('Failed') if worlds.where(name: values[:name], zone: values[:zone]).exists?
  end
end
```

**整理表单对象**

我们只需要添加一些方法来检查属性和创建/更新对象。

to_hash 方法使用属性及其值创建散列。在调用 world contract . new . call(attributes)之后，我们将在模式对象中得到验证结果。如果模式包含错误，那么我修改它们以便更好地呈现(你可以有别的东西)并且 save 方法返回 false。如果验证通过，那么我通过设置表单对象的属性来创建/更新世界对象，并保存。

```
class WorldForm < Dry::Struct
  ... attr_reader :world, :errors def save
    attributes = to_hash
    schema = WorldContract.new.call(attributes)
    [@errors](http://twitter.com/errors) = schema.errors(locale: I18n.locale).to_h.values.flatten
    return false unless errors.empty?
    [@world](http://twitter.com/world) = attributes[:id] ? World.find_by(id: attributes[:id]) : World.new
    world.attributes = attributes.except(:id)
    world.save
  end
end
```

**添加 I18n**

您可以将本地化参数传递给契约类。

```
class WorldContract < Dry::Validation::Contract
  config.messages.namespace = :world
  config.messages.backend = :i18n ... rule(:id, :name, :zone) do
    ... key.failure(I18n.t('dry_validation.errors.world.is_exist')) if worlds.where(name: values[:name], zone: values[:zone]).exists?
  end
end
```

并使用本地化创建新 yaml 文件。

```
en:
  dry_validation:
    errors:
      world:
        is_exist: World with such params already exists
        rules:
          name:
            filled?: Name can't be blank
          zone:
            filled?: Zone can't be blank
```

通过在 save 方法中调用 schema . errors(locale:i18n . locale ),您将在 WorldForm 对象的 errors 变量中找到本地化的错误。

**结果**

所以用 dry-rb 完成的表单对象看起来像这样。

```
class WorldForm < Dry::Struct
  module Types
    include Dry::Types(default: :nominal)
  end attribute :id, Types::Integer.optional
  attribute :name, Types::String
  attribute :zone, Types::Integer attr_reader :world, :errors def save
    attributes = to_hash
    schema = WorldContract.new.call(attributes)
    [@errors](http://twitter.com/errors) = schema.errors(locale: I18n.locale).to_h.values.flatten
    return false unless errors.empty?
    [@world](http://twitter.com/world) = attributes[:id] ? World.find_by(id: attributes[:id]) : World.new
    world.attributes = attributes.except(:id)
    world.save
  end
endclass WorldContract < Dry::Validation::Contract
  config.messages.namespace = :world
  config.messages.backend = :i18n schema do
    optional(:id)
    required(:name).filled(:string)
    required(:zone).filled(:string)
  end rule(:id, :name, :zone) do
    worlds = values[:id].nil? ? World.all : World.where.not(id: values[:id]) key.failure(I18n.t('dry_validation.errors.world.is_exist')) if worlds.where(name: values[:name], zone: values[:zone]).exists?
  end
end
```

**使用表单对象**

然后你可以在控制器或交互器中使用你的表单对象。

为了用表单对象创建世界，我传递了带符号键和 nil id 的参数。成功保存后，您可以使用 world_form.world 来呈现 json 或其他任何内容。以及保存失败后 world _ form.errors 为渲染错误。

```
world_form = WorldForm.new(world_params.merge(id: nil))
if world_form.save
  # world_form.world
  ...
else
  # world_form.errors
  ...
end...def world_params
  params.require(:world).permit(:name, :zone).to_h.symbolize_keys
end
```

为了更新世界对象，我只改变了世界形状的属性。

```
world_form = WorldForm.new([@world](http://twitter.com/world).attributes.symbolize_keys.merge(world_params))
if world_form.save
  ...
```

**自定义类型**

如果您需要验证与其他模型的关系，您将需要添加一些额外的代码。

```
class ActivityForm < Dry::Struct
  module Types
    ...
    World = Dry::Types::Nominal.new(::World)
  end ...
  attribute :world, Types::World
  ...
endclass ActivityContract < Dry::Validation::Contract schema do
    ...
    required(:world).filled
  end
end
```

**用 RSpec 测试**

我们可以用 rspec 测试表单对象。

```
RSpec.describe WorldForm, type: :service do
  describe '.save' do
    context 'for invalid data' do
      let(:service) { described_class.new(id: nil, name: '', zone: '') } it 'does not create new world' do
        expect { service.save }.to_not change(World, :count)
      end it 'and returns false' do
        expect(service.save).to eq false
      end it 'and form contains errors' do
        service.save expect(service.errors.size.positive?).to eq true
      end
    end context 'for valid data' do
      let!(:world) { create :world } context 'for existed world' do
        let(:service) { described_class.new(id: nil, name: world.name, zone: world.zone) } it 'does not create new world' do
          expect { service.save }.to_not change(World, :count)
        end it 'and returns false' do
          expect(service.save).to eq false
        end it 'and form contains errors' do
          service.save expect(service.errors.size.positive?).to eq true
        end
      end context 'for unexisted world' do
        let(:service) { described_class.new(id: nil, name: 'Хроми', zone: 'RU') } it 'creates new world' do
          expect { service.save }.to change { World.count }.by(1)
        end it 'and returns true' do
          expect(service.save).to eq true
        end it 'and form contains created world' do
          service.save expect(service.world).to eq World.last
        end
      end
    end context 'for updating' do
      let!(:world1) { create :world }
      let!(:world2) { create :world } context 'for invalid data' do
        let(:service) { described_class.new(id: world1.id, name: '', zone: '') } it 'does not update world' do
          service.save
          world1.reload expect(world1.name).to_not eq ''
          expect(world1.zone).to_not eq ''
        end it 'and form contains errors' do
          service.save expect(service.errors.size.positive?).to eq true
        end
      end context 'for existed data' do
        let(:service) { described_class.new(id: world1.id, name: world2.name, zone: world2.zone) } it 'does not update world' do
          service.save
          world1.reload expect(world1.name).to_not eq world2.name
        end it 'and form contains errors' do
          service.save expect(service.errors.size.positive?).to eq true
        end
      end context 'for valid data' do
        let(:service) { described_class.new(id: world1.id, name: 'Хроми', zone: 'RU') } it 'does not update world' do
          service.save
          world1.reload expect(world1.name).to eq 'Хроми'
          expect(world1.zone).to eq 'RU'
        end it 'and returns true' do
          expect(service.save).to eq true
        end it 'and form contains updated world' do
          service.save expect(service.world.id).to eq world1.id
        end
      end
    end
  end
end
```

**结论**

对表单对象使用 **dry-rb** 允许你干净地保存你的模型，并把你所有的验证逻辑放在一个明确界定的地方。