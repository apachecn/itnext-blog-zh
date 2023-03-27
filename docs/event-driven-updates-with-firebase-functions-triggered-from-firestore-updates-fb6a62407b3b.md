# 从 Firestore 更新中触发具有 Firebase 功能的事件驱动更新

> 原文：<https://itnext.io/event-driven-updates-with-firebase-functions-triggered-from-firestore-updates-fb6a62407b3b?source=collection_archive---------3----------------------->

![](img/b4a7fb7e6787c96f706a0a002d4e0ae9.png)

想象一个典型的场景，用户将从前端发起一个动作，该动作将触发对管理数据集的更新。当然，您可以在操作过程中直接进行更新，但这可能会导致更长的操作时间，甚至导致错误。

> 保持用户界面/UX 尽可能的流畅和快速应该是一个优先考虑的问题，因此最好是只为用户做最不必要的动作，异步地做任何其他的任务。

这种管理任务的一些例子可以是给定项目的等级的重新计算，或者是 [NPS](https://www.hotjar.com/net-promoter-score/) 分数。

# 前端代码——React 或任何其他框架

在这种情况下，我们将重点关注 NPS 示例。

```
import app from "firebase/app";
import "firebase/firestore";
class Firebase {
  constructor() {
    app.initializeApp(config);
    this.store = app.firestore();
  }
  trackNPSVote = async ({ score, comment = "" }) => {
    const NPSRef = this.store.collection("NPS");
    let autoID = await NPSRef.doc().id;
    const data = {
      score,
      createdAt: new Date().toISOString(),
      comment,
    };
    return await NPSRef.doc(`${autoID}`).set(data);
  };
}
export default Firebase;
```

当然，需要调用这段代码，想象一个简单的 UI，有 1-10 个按钮和一个可选的注释字段，在提交时，调用 trackNPSVote 函数。

# 后端代码—云功能

现在，我们需要监听用户在我们的云函数中创建的新文档(NPS 条目)

```
const functions = require("firebase-functions");
function calculateNPS(scores) {
  var promoters = 0;
  var detractors = 0;
  for (var i = 0, l = scores.length; i < l; i++) {
    if (scores[i] >= 9) promoters++;
    if (scores[i] <= 6) detractors++;
  }
  return Math.round((promoters / l - detractors / l) * 100);
}
exports.onNPSQuestionnaireFilled = functions.firestore
  .document("NPS/{npsEntry}")
  .onCreate(async (snap) => {
    const questionnaire = snap.data();
    const npsRef = firestore.collection("adminCollection").doc("NPS");
    const doc = await npsRef.get();
    try {
      // we already have some historic data
      if (doc.exists) {
        const npsStats = doc.data();
        npsStats.scores.push(questionnaire.score);
        npsStats.score = calculateNPS(npsStats.scores);
        return await npsRef.set(npsStats);
      } else {
        // we need to create the document
        const score = calculateNPS([questionnaire.score]);
        const data = {
          score: score,
          scores: [questionnaire.score],
        };
        await npsRef.set(data);
      }
      return true;
    } catch (err) {
      console.error(err.message);
    }
  });
```

现在，我们可以计算我们的 NPS 或任何其他任意 KPI，而不会对我们的用户造成任何干扰。