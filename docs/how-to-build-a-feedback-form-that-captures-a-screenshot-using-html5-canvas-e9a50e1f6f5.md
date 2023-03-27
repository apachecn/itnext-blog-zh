# 如何使用 HTML5 Canvas 构建一个捕捉屏幕截图的反馈表单

> 原文：<https://itnext.io/how-to-build-a-feedback-form-that-captures-a-screenshot-using-html5-canvas-e9a50e1f6f5?source=collection_archive---------9----------------------->

我有一个客户，他的应用程序得到反馈，从外部 API 返回的一些数据与预期不符。在查看日志并试图找出问题后，我决定我们需要获取令人不快的结果的屏幕截图。让应用程序的最终用户手动完成这一过程可能会让我们得不到我们所需要的东西。是时候实现自动化了！

该应用的后端是用 Rails 编写的。我想任何后端都可以做到这一点；它甚至可以是无服务器的。我们的目标是:

*   无需用户交互即可捕获应用程序的屏幕截图
*   显示一个要求简短和详细描述的表单
*   提交时，获取信息并通过电子邮件发送给合适的人

让我们深入到它的后端！

首先，我需要一个端点来发布我的表单，所以我在 config/routes.rb 中创建了一个条目:

```
post '/feedback' *=>* 'feedback#create', constraints: { format: 'json' }
```

然后我创建了`FeedbackController`:

```
**class** *FeedbackController* < ApplicationController

  **def** create
    img = feedback_params[:img]
    subj = feedback_params[:subject]
    desc = feedback_params[:description]
    FeedbackMailer.email_admins(img, subj, desc).deliver_later
    render json: **nil**, status: :ok
  **end** protected

  **def** feedback_params
    params.require(:feedback).permit(%i[img subject description])
  **end
end**
```

我想这是不言自明的。从请求中获取`params`，并将它们发送到我用`rails g mailer Feedback`创建的`FeedbackMailer`类。

```
**class** *FeedbackMailer* < ApplicationMailer

  **def email_**admins(*img*, *subj*, *desc*)
    @img = *img* @subject = *subj* @description = *desc
    mail*(to: ENV['FEEDBACK_EMAIL_RECEIVER'], subject: 'Sorting Hat feedback')
  **end
end**
```

这个特殊的应用程序以前从未发送过电子邮件，所以我们需要为它抓取一个响应的布局模板。由于它是针对内部用户，我们希望它看起来不错，但它不需要赢得设计奖。一个好的入门模板可以在这里找到:[https://github.com/leemunroe/responsive-html-email-template](https://github.com/leemunroe/responsive-html-email-template)

对于 Rails，我简单地从这个模板中删除了起始副本，添加了一个`<%= yield %>` erb 标签，我们就可以开始工作了。

对于实际的电子邮件内容，我们基本上是吐出`FeedbackMailer`收到的参数:

```
<h1><%= @subject %></h1>

<p><%= @description %></p>

<p>Here's a picture of what they had on screen:</p>

<small>(note this tool does not capture product images)</small>
<img src="<%= @img %>" alt="screenshot">
```

开发的最后一步是确保我们可以模拟获取电子邮件，为此我使用了优秀的`mailcatcher` gem。然后在`config/environments/development.rb`中，我添加了捕获电子邮件的 SMTP 配置:

```
*# gem install mailcatcher* config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = { address: 'localhost', port: 1025 }
```

对于生产，我们想做几件事。异步处理电子邮件很重要，因为最终用户最终会等待服务器处理发送电子邮件，谁知道这是否可行！我们选择使用 sidekiq 和 Redis 来处理后台工作，这需要我们的`config/environments/production.rb`中的一些条目:

```
*# Use a real queuing backend for Active Job (and separate queues per environment)* config.active_job.queue_adapter = :sidekiq
config.active_job.queue_name_prefix = **"appname_**#{Rails.env}**"** config.action_mailer.perform_caching = **false**config.action_mailer.default_url_options = { host: 'myapp.com' }
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
    address: ENV['SMTP_HOST'],
    port: ENV['SMTP_PORT'],
    username: ENV['SMTP_USER'],
    password: ENV['SMTP_PASSWORD']
}
```

这是我们的后端设置！对于 Rails 应用程序来说，没有什么疯狂或不标准的。

对于前端，我们想添加一个按钮，它将捕获在中呈现的内容并呈现一个模态。是时候做些 HTML 了！

```
<button class="btn btn-secondary" id="feedback">
  <span id="feedbackSpinner" class="spinner"></span>
  Feedback
</button>
```

我们的模式:

```
<div id="feedbackModal" class="modal" tabindex="-1" role="dialog">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Send Feedback</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <p>Let us know if something is incorrect, needs improvement, or any other general feedback about Sorting Hat.</p>
        <div class="form-group">
          <label for="subject">Summary</label>
          <input id="subject" type="text" class="form-control" placeholder="Brief description, name, email, etc.">
        </div>
        <div class="form-group">
          <label for="description">Description</label>
          <textarea id="description" row="3" class="form-control" placeholder="What were you searching for? What was expected? What happened?"></textarea>
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
        <button  id="sendFeedback" type="button" class="btn btn-primary">Send Feedback</button>
      </div>
    </div>
  </div>
</div>
```

好了，这里是标准的引导模式和按钮组件。现在来看看 Javascript。我们找到了一个名为 [html2canvas](https://github.com/niklasvh/html2canvas) 的很棒的库来帮助我们捕捉屏幕。它基于承诺，这使得延迟显示模态非常容易，直到屏幕截图完成。因为一直拍模特的照片是不合适的。html2canvas 可以通过选择器名称捕获任何内容，这使得它非常灵活，甚至可以用于图像处理工具或任何其他您可以想象的疯狂功能。在我们的例子中，我们只是发布屏幕截图的数据 uri，因为没有理由在我们的服务器上长期存储屏幕截图。

```
(**function** ($) {
  $(document).on('turbolinks:load', **function** () {

    **let** img = **null**;

    **function** sendFeedback() {
      **let** subject = $('input#subject').val()
      **let** description = $('input#description').val()
      $.ajax({
        beforeSend: **function**(xhr) {xhr.setRequestHeader('X-CSRF-Token', $('meta[name="csrf-token"]').attr('content'))},
        url: '/feedback',
        method: 'post',
        data: {
          feedback: {
            subject: subject,
            description: description,
            img: img
          }
        },
        success(e) {
          $feedbackModal.modal('hide')
          tmpl = $("#alert").html();
          $.results.html(tmpl({error: "Thanks! We received your feedback and will get back to you soon.", type: 'success'}));
        }
      })
    }
    $.itemSpinner = Handlebars.compile($('#item-spinner').html());
    **function** startItemSpinner(el) {
      $(el).html($.itemSpinner);
    }

    **function** stopItemSpinner(el) {
      $(el).html('');
    }

    **let** $feedbackModal = $('#feedbackModal').modal({show: **false**});

    $('button#feedback').click(**function** (evt) {
      evt.preventDefault();
        startItemSpinner('#feedbackSpinner');
      html2canvas(document.body).then(**function** (canvas) {
        img = canvas.toDataURL();
        stopItemSpinner('#feedbackSpinner');
        $feedbackModal.modal('show')
      });
    })

    $('button#sendFeedback').click(sendFeedback)
  })
})(jQuery);
```

是的*喘息*它使用的是 jQuery，而 alert 和 spinner 就是这一点车把模板:

```
<script id="item-spinner" type="text/x-handlebars-template">
  <i class="fa fa-spin fa-cog"></i>
</script><script id="alert" type="text/x-handlebars-template" charset="utf-8">
  <div class="alert alert-{{type}} alert-dismissible fade show" role="alert">
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>
    {{error}}
  </div>
</script>
```

你可以随意修改。这个特定的应用程序已经有了这些库，没有理由围绕它构建一个巨大的同构 node.js ui。符合您的需求！

主要的逻辑是这样的:

```
html2canvas(document.body).then(**function** (canvas) {
        img = canvas.toDataURL();
        stopItemSpinner('#feedbackSpinner');
        $feedbackModal.modal('show')
      });
```

我发现处理屏幕截图可能需要 1-2 秒。只要足够长，我们需要提供一些反馈，表明它正在工作。因此，我们展示了一个微调，直到模态显示。然后，我们有一个“发送反馈”按钮的处理程序，向我们的控制器发送一个 AJAX 调用，并开始这个过程。这里不需要多部分，因为我们使用的是`canvas.toDataUR()`方法，它给出了图像的 URI 表示。

这就是我们所有的代码。您应该启动 mailcatcher，看看是否能得到反馈！

想给我一些反馈吗？对用其他语言或变体实现有疑问吗？给我发消息！感谢阅读！