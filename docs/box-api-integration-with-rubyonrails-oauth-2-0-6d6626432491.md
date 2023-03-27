# BOX API 与 RubyOnRails 的集成(OAuth 2.0)

> 原文：<https://itnext.io/box-api-integration-with-rubyonrails-oauth-2-0-6d6626432491?source=collection_archive---------0----------------------->

![](img/f57eb66db2f6eca46cbee11a33afd49a.png)

**使用 OAuth 2.0 登录**

[](https://www.box.com/) [## 安全的文件共享、存储和协作|盒子

### 从简单的文件共享到构建定制应用，Box 正在改变您管理整个企业内容的方式。

www.box.com](https://www.box.com/) 

框|为企业提供安全的内容和在线文件共享

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fbox-api-integration-with-rubyonrails-oauth-2–0–6d6626432491) 上分享这篇文章

Box 为个人、团队和企业提供安全的内容管理和协作，实现安全的文件共享和在线文件访问。

OAuth2.0 请参考以下链接:
[http://www . bubble code . net/en/2016/01/22/understanding-oauth 2/](http://www.bubblecode.net/en/2016/01/22/understanding-oauth2/)

要配置 BOX 中的应用程序，请按照下列步骤操作:

**第一步:**创建开发者账户

[https://app.box.com/signup/o/default_developer_offer](https://app.box.com/signup/o/default_developer_offer)

如果你忽视了。

**步骤 2:** 遵循以下链接中的步骤:

[https://box-content.readme.io/docs/oauth-20](https://box-content.readme.io/docs/oauth-20)

**第三步:**完成 BOX 中的 app 设置后，创建一个 rails 应用。让我们假设，控制器名是 box_api_controller.rb

在 box_api_controller.rb 文件中，

# 登录时创建请求按钮

```
def make_request
    #Check access token expire or not.
    check_access_token_expire = check_access_token_expire_dt
    if check_access_token_expire.split("-")[0] == "access_token"
        #Create client by passing Token
        @box_client = Boxr::Client.new(check_access_token_expire.split("-")[1])
        cookies[:token] = check_access_token_expire.split("-")[1]
    else
        if check_access_token_expire.split("-")[0] == "refresh_token"
            #Call method
            create_post_req_url("refresh_token","refresh_token",check_access_token_expire.split("-")[1])
        else
            # kick off authorization flow
            parameters = "response_type=code&client_id=<your client id>&redirect_uri=<your application url>/handle_user_decision/&state=security_token"
            url = "https://account.box.com/api/oauth2/authorize?#{parameters}"
            redirect_to url
        end
    end end

##After authorized the client id, get code in response
    def handle_user_decision
    # kick off authorization flow
    #Get authorization code
    code_url = Rack::Utils.parse_query URI(request.original_url).query
    code = code_url["code"] 
    #Call method
    create_post_req_url("authorization_code","code", code) 
end
```

# 创建帖子 URL

```
def create_post_req_url(grant_type,header, code)
    #Set oauth2 url
    uri = URI.parse("https://api.box.com//oauth2//token")
    #Passing parameter
    data = "grant_type=#{grant_type}&#{header}=#{code}&client_id=<your client id>&client_secret=<your client secret key>"
    #Set header
    headers = {"Content-Type" => "application/x-www-form-urlencoded"}
    #Get http request
    http = Net::HTTP.new(uri.host,uri.port)
    http.use_ssl = true
    http.verify_mode = OpenSSL::SSL::VERIFY_NONE
    #Do post the URL
    response = http.post(uri.path,data.to_s,headers)
    #Check response
    if response.code != "200"
        flash[:alert] ="エラーが発生しました。管理者に連絡してください。エラーの内容：#{response.code}  #{JSON.parse(response.body)}"
    else
        #flash[:alert] ="#{response.body.to_json}"
        parsed = JSON.parse(response.body) # returns a hash
        token = parsed["access_token"]
        cookies[:token] = nil
        cookies[:token] = token      
        if grant_type == "authorization_code"
            #Insert BOX access token details
            user = "<your drive user name>"
            insert_access_token(user, token, parsed["refresh_token"], Time.now)
        else
            if grant_type == "refresh_token"
                #Update BOX access token 
                updt_access_token(user, token, code, parsed["refresh_token"], Time.now)
            end  
        end
        redirect_to box_api_index_path
    end
end
```

其他辅助方法:-

# 检查访问令牌是否过期。

```
def check_access_token_expire_dt
    @access_token_time = BoxApiAccessToken.getaccesstokentime
    if !@access_token_time.blank?
        @access_token_time.each do |token_details |
            if token_details.access_token_dt != nil
                if token_details.new_access_token_dt.to_datetime.new_offset(Rational(9, 24)).strftime('%Y/%m/%d %H:%M') < Time.now.to_datetime.new_offset(Rational(9, 24)).strftime('%Y/%m/%d %H:%M')
                    check_access_token_expire_dt = "refresh_token-#{token_details.refresh_access_token}"
                    return check_access_token_expire_dt
                else
                    check_access_token_expire_dt = "access_token-#{token_details.access_token}"
                    return check_access_token_expire_dt
                end
            else
                check_access_token_expire_dt = "new_token-req_new_token"
                return check_access_token_expire_dt
            end
        end
    else
        check_access_token_expire_dt = "new_token-req_new_token"
        return check_access_token_expire_dt
    end
end
```

# 在数据库中插入访问令牌详细信息

```
def insert_access_token(user,access_token,refresh_access_token,access_token_dt)
    @box_access_token = BoxApiAccessToken.new(
            :user => user,
            :access_token => access_token,
            :refresh_access_token => refresh_access_token,
            :access_token_dt => access_token_dt)

            #Save User Device Data
            @box_access_token.save
end

#Update access_token,refresh_access_token,access_token_dt details in DB
    def updt_access_token(user,access_token, refresh_access_token,new_refresh_access_token,access_token_dt)
    #@box_access_token_updt = BoxApiAccessToken.find_refresh_access_token(refresh_access_token)
    @box_access_token_updt = BoxApiAccessToken.find_by_refresh_access_token(refresh_access_token)
    attributes = {:access_token => access_token,:access_token_dt => access_token_dt, :refresh_access_token => new_refresh_access_token, :updated_at => access_token_dt}
    #Update the object
    @box_access_token_updt.update_attributes(attributes)
end
```

在模型中，

```
class BoxApiAccessToken < ActiveRecord::Base
    scope :getaccesstokentime, lambda { 
select("id,access_token,refresh_access_token,substring(cast(access_token_dt as varchar),0,17) as access_token_dt,substring(cast(access_token_dt + interval '60 minute' as varchar),0,17) as new_access_token_dt ").order(:id => "desc").limit(1)
    }   
end
```

# 在 index.html.erb 文件中

```
<%= form_tag(:controller => "box_api", :action => 'make_request') do |f| %>
<div class="form-group"><%= submit_tag("Box Login", class: "btn btn-primary") %></div><% end %>
```

# 这是我的迁移文件

```
class CreateBoxApiAccessTokens < ActiveRecord::Migration
    def change
        create_table :box_api_access_tokens do |t|
            t.string :user, :limit => 100, :null => false
            t.string :access_token, :limit => 500
            t.string :refresh_access_token, :limit => 500
            t.datetime :access_token_dt, :limit => 50
            t.datetime :created_at, :null => false
            t.string :created_by, :limit => 50
            t.datetime :updated_at
            t.string :updated_by, :limit => 50
            t.boolean :del_flag, :default => false      
            end
        execute 'alter table box_api_access_tokens alter column created_at set default now()'
    end
end
```

请注意，使用访问和刷新令牌时，access_token 是发出 API 请求所需的实际字符串。每个 access_token 的有效期为 1 小时。为了获得新的有效令牌，您可以使用附带的 refresh_token。每个 refresh_token 在 60 天内只能使用一次。每次您使用 refresh_token 获得新的 access_token 时，我们都会将您的计时器重置为 60 天，并发给您一个新的 refresh_token。

这意味着只要您的用户每 60 天使用一次您的应用程序，他们的登录就永远有效。

有关更多详细信息，请参考 BOX API 文档。

享受编码。

## 感谢并致以最诚挚的问候

*原载于【qiita.com】[](https://qiita.com/alokrawat050/items/3028c89b9e449a6f614e)**。***