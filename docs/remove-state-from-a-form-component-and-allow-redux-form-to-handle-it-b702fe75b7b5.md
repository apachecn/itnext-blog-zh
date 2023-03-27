# 从表单组件中移除状态，并允许 Redux-Form 处理它

> 原文：<https://itnext.io/remove-state-from-a-form-component-and-allow-redux-form-to-handle-it-b702fe75b7b5?source=collection_archive---------6----------------------->

当我在 Redux 应用程序中使用 React form 组件来控制它自己的本地状态时，我可以感觉到干草叉正在变尖。去纯粹主义者那里，这不是惯例吗？你要么允许 Redux 的商店管理你的应用程序的整个状态，或者你根本没有！没有中间地带。对吗？

这正是我在决定如何处理组件状态时所处的困境。我还不是 100%确定我需要在我的应用程序状态中包含什么。所以我开始让它保持独立。这有助于防止我不得不写不必要的动作，减少，然后担心回去，改变或删除的东西在未来。

让我先给你看看我想出来的，然后我们可以解决我们的 Redux 问题。

```
import React, { Component } from 'react';
import { Form, Message, Modal } from 'semantic-ui-react';
import ModalPlayerButtons from './ModalPlayerButtons';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import { currentPlayer } from '../actions';class QuestionModalClue extends Component {
  constructor(props) {
    super(props);this.state = {
      disabled: true,
      hidden: true,
      correct: false,
      value: ''
    }
    this.playersAnswer = this.playersAnswer.bind(this);
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

 handleChange(e) {
    this.setState({
      value: e.target.value
    });
  }handleSubmit(e) {
    e.preventDefault();
    this.setState({
      hidden: false,
      value: ''
    })
    if (this.state.value.toLowerCase() === this.props.clue.answer.toLowerCase()) {
      this.setState({
        correct: true
      })
    }
    this.setState({
      disabled: true
    })
  }playersAnswer = (e) => {
    let player = e.currentTarget.textContent;
    this.props.currentPlayer(player);
    this.setState({
      disabled: false
    })
  }render() {
    return (
      <Modal 
        open={this.props.modalOpen}
        centered={true}
        size='fullscreen' 
        dimmer='blurring'
        style={{ 'textAlign': 'center', 'marginTop': '20px', 'backgroundColor': '#2185d0' }} 
      >
        <Modal.Content
          style={{ 'backgroundColor': '#2185d0', 'color': '#fff' }}
        >
          {this.props.clue !== undefined &&
            <Modal.Header
              as='h1'
              onClick={this.props.test}      
            >
              {this.props.clue.description}
            </Modal.Header>
          }
        </Modal.Content>
        <ModalPlayerButtons playersAnswer={this.playersAnswer} />

        <Form
          as='form'
          size='massive'
          onSubmit={this.handleSubmit}
        >
          <Form.Field width={6} style={{ 'marginLeft': 'auto', 'marginRight': 'auto', 'marginTop': '20px', 'marginBottom': '10px' }}
            disabled={this.state.disabled}
          >
            <label style={{'color': 'white'}}>Player's Answer</label>
            <Form.Input 
              type="text"
              placeholder="What is..."
              value={this.state.value}
              onChange={this.handleChange}
            />
          </Form.Field>
          <Form.Field
            width={6}
            style={{ 'marginLeft': 'auto', 'marginRight': 'auto', 'marginTop': '20px', 'marginBottom': '10px' }}
          >
          {
            this.state.correct ? (
              <Message
                hidden={this.state.hidden}
                size="massive"
                color="green"
                header="Correct Answer"
                content="Current score plus 200"
              />
            ) : (
              <Message
                hidden={this.state.hidden}
                size="massive"
                color="red"
                header="Incorrect Answer"
                content="Current score minus 200"
              />
            )
          }
          </Form.Field>
        </Form>
      </Modal>
    )
  }
}const mapDispatchToProps = dispatch => {
  return bindActionCreators({
    currentPlayer
  }, dispatch);  
}
export default connect(null, mapDispatchToProps)(QuestionModalClue);
```

更好的是，也可能更糟的是，我同时还连接到了我的组件中的 Redux 的 store。我们有一些功能只是控制状态，其他的控制状态和来自 Redux 的道具。到处都是。在这个组件中，我有两个不同的东西来控制我的状态。谈谈调试和效率的噩梦。我能感觉到暴民在追杀我。

那我该怎么办呢？看着状态中的不同项目，我不想担心在 Redux 中控制它们，并手动编码每个单独的项目。我订阅编码员都很懒，那就找点东西替我们管管这些东西吧。但我必须指出这一点，我也坚信知道幕后发生了什么。所以要知道你的工具在为你做什么。

这让我想到了这个博客的标题。让我们来谈谈“还原形式”。Redux-Form 使状态变得简单。Redux-Form 将自己描述为在 Redux 中管理表单状态的最佳方式。

这个博客并不是设置 Redux-Form 的完整教程。但是他们在他们的网站上给出了一个很好的教程，将 Redux-Form 包含在一个项目中。

快速地，如果你已经有一个制作好的项目，只需要用`yarn add redux-form`包含 Redux-Form。如果你没有注意到，我将使用`yarn`而不是`npm`。按照说明将`redux-form`连接到您的 redux 商店。然后只需遵循他们的 api 来包含您的必要项目。

所以让我们再来看看同一个组件，但是这次允许`redux-form`控制我以前的状态。

```
import React, { Component } from 'react';
import { Form, Message, Modal } from 'semantic-ui-react';
import ModalPlayerButtons from './ModalPlayerButtons';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import { currentPlayer } from '../actions';
import { Field, reduxForm, formValueSelector } from 'redux-form';class QuestionModalClue extends Component {
  constructor(props) {
    super(props); // variable names to be assigned depending on result of answer
    this.correctAnswer;
    this.incorrectAnswer; this.playersAnswer = this.playersAnswer.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  } handleSubmit(e) {
    e.preventDefault();
    if (this.props.answerValue.toLowerCase() === this.props.clue.answer.toLowerCase()) {
      this.correctAnswer = <Message
        size="massive"
        color="green"
        header="Correct Answer"
        content="Current score plus 200"
      />
      this.forceUpdate();
    } else {
      this.incorrectAnswer = <Message
        size="massive"
        color="red"
        header="Incorrect Answer"
        content="Current score minus 200"
      />
      this.forceUpdate();
    }
  } playersAnswer = (e) => {
    let player = e.currentTarget.textContent;
    this.props.currentPlayer(player);
  } render() { const { answerValue } = this.props; return (
      <Modal 
        open={this.props.modalOpen}
        centered={true}
        size='fullscreen' 
        dimmer='blurring'
        style={{ 'textAlign': 'center', 'marginTop': '20px', 'backgroundColor': '#2185d0' }} 
      >
        <Modal.Content
          style={{ 'backgroundColor': '#2185d0', 'color': '#fff' }}
        >
          {
            this.props.clue !== undefined &&
              <Modal.Header
                as='h1'
                onClick={this.props.modalClose}      
              >
                {this.props.clue.description}
              </Modal.Header>
          }
        </Modal.Content>
        <ModalPlayerButtons playersAnswer={this.playersAnswer} />
        <Form
          as='form'
          size='massive'
          onSubmit={this.handleSubmit}
        >
          <Form.Field 
            width={6} 
            style={{ 'marginLeft': 'auto', 'marginRight': 'auto', 'marginTop': '20px', 'marginBottom': '10px' }}
            disabled={this.props.playerGuessing == undefined}
          >
            <label style={{'color': 'white'}}>Player's Answer</label>
            <Field 
              name='playerAnswer' 
              component='input' 
              type='text' 
              placeholder='What is... ?' 
            />
          </Form.Field>
          <Form.Field
            width={6}
            style={{ 'marginLeft': 'auto', 'marginRight': 'auto', 'marginTop': '20px', 'marginBottom': '10px' }}
          >
            {this.correctAnswer}
            {this.incorrectAnswer}
          </Form.Field>
        </Form>
      </Modal>
    )
  }
}const mapStateToProps = state => {
  return {
    playerGuessing: state.currentPlayer.currentPlayer
  }
}const mapDispatchToProps = dispatch => {
  return bindActionCreators({
    currentPlayer
  }, dispatch);  
}QuestionModalClue = reduxForm({
  form: 'answer'
})(QuestionModalClue);const selector = formValueSelector('answer');QuestionModalClue = connect(state => {
  const answerValue = selector(state, 'playerAnswer')
  return {
    answerValue,
  }
})(QuestionModalClue);export default connect(mapStateToProps, mapDispatchToProps)(QuestionModalClue);
```

你可能注意到的第一件事是 state 已经从我们的构造函数中移除了。我们的函数不再用于设置组件状态。相反，它们被用来给我们的变量赋值。我创建了一个新的`mapStateToProps`函数来访问我们的 redux 存储。这是为了当玩家被点击时，表单输入会显示。在代码的底部有点混乱。所以让我们稍微回顾一下。

```
QuestionModalClue = reduxForm({
  form: 'answer'
})(QuestionModalClue);
```

这用于将我们的表单直接连接到 redux。我们使用`form: 'answer'`来命名我们的表单以供参考。

然后是`const selector = formValueSelector('answer');`。我们可以使用`formValueSelector`来连接表单值。它*创建*一个选择器函数，接受字段名并从指定的表单返回相应的值。参考 Redux-Form 站点，因为有几种不同的方法来实现这一点。您可能不需要一个单独的`mapStateToProps`函数，或者您可能在一个字段组中有多个想要访问的字段。

```
QuestionModalClue = connect(state => {
  const answerValue = selector(state, 'playerAnswer')
  return {
    answerValue
  }
})(QuestionModalClue);
```

在这里，我们可以选择使用表单中输入名称的值来分配变量名。然后我们返回这个值，这样我们就可以访问它了。

```
export default connect(mapStateToProps, mapDispatchToProps)(QuestionModalClue);
```

最后，我们通常连接到 redux，这使我们能够使用`mapStateToProps`和`mapDispatchToProps`。你可以选择将其中的一些组合在一起，但对我来说，为了可读性更好，更容易将它们分开。

现在每个人都可以放松了。我的组件中不再包含本地状态，但它们都在 redux 的存储中。我就不说什么是正确的，什么是不正确的了。但是我完全理解在组件内部控制简单表单的本地状态，而不是使用 redux。我很想在下面的评论中听到一些观点或者任何其他关于控制状态的建议。我的博客到此结束。我想这是我的下一个项目或想法。下次见。