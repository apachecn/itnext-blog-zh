# Promise 用 Three.js 加载

> 原文：<https://itnext.io/promise-loading-with-three-js-78a6297652a5?source=collection_archive---------2----------------------->

这是一篇关于我最近使用(原生)ES6 承诺简化 Three.js 项目中加载处理的经验的快速帖子。

![](img/f1bd50b805e531ee862f559ec42eaac7.png)

通过 Three.js 示例，加载几何体和相应纹理的常见模式如下所示:

```
const material = new THREE.MeshStandardMaterial({
  map: new THREE.TextureLoader().load('map.jpg'),
  normalMap: new THREE.TextureLoader().load('normalMap.jpg')
});const loader = new THREE.JSONLoader();loader.load('geometry.json', geometry => {
  const mesh = new THREE.Mesh(geometry, material);

  scene.add(mesh);
});
```

`TextureLoader.load()`返回一个纹理，当图像加载时更新。向`JSONLoader.load()`传递一个 onComplete 回调，在 JSON 被加载和处理时调用这个回调。当回调被调用时，一个网格被创建，无论纹理是否被加载。

有时这种行为是需要的；你要尽快向*展示一些东西*。但也有一些问题，我相信你们以前都见过。如果几何图形在纹理之前加载，在纹理一个接一个弹出之前，模型看起来是无纹理的，*裸露的*。当它们被处理并上传到 GPU 时，会有明显的丢帧现象。

幸运的是，使用[承诺](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)很容易解决这些问题。

首先，让我们围绕一个`JSONLoader`和一个`Promise`创建一个包装函数。

```
function loadJSON(url) {
  return new Promise(resolve => {
    new THREE.JSONLoader.load(url, resolve);
  });
}
```

该函数返回一个承诺，在加载文件时，该承诺将与加载的几何图形一起解析。

接下来我们可以在一个`TextureLoader`周围做一个类似的包装。

```
function loadJSON(url) {
  return new Promise(resolve => {
    new THREE.TextureLoader().load(url, resolve);
  });
}
```

请注意，`TextureLoader`也有一个类似于`TextureLoader`的 onComplete 回调，尽管使用频率较低。事实上，我非常确定所有三个. js 加载器都符合这个约定。

现在让我们围绕纹理承诺编写另一个包装器，它将通过一个 Material 实例来解决。

```
function loadMaterial() {
  const textures = {
    map: 'map.jpg',
    normalMap: 'normalMap.jpg'
  };

  const params = {};

  const promises = Object.keys(textures).map(key => {
    return loadTexture(textures[key]).then(texture => {
      params[key] = texture;  
    });
  });

  return Promise.all(promises).then(() => {
    return new THREE.MeshStandardMaterial(params);
  });
}
```

`[Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)`接受一个承诺数组，并返回一个承诺，该承诺在所有子承诺都解析后解析。

接下来，我们可以将几何和材料承诺包装器组合成一个最终的 majestic 包装器函数，再次依赖于`Promise.all()`。

```
function loadMesh() {
  const promises = [
    loadGeometry(),
    loadMaterial()
  ];

  return Promise.all(promises).then(result => {
    return new THREE.Mesh(result[0], result[1]);
  });
}
```

`Promise.all()`以与您提供的承诺(或值)顺序相同的值数组进行解析。在这种情况下，`result`数组看起来像`[geometry, material]`。

使用上面的代码，我们基本上创建了一个迷你加载管理器，一旦所有的资产都被加载，它就创建一个网格，所有的都使用很少的(本地)JavaScript。干净利落。

让我们再充实一点。下面的代码只是实现这一点的许多可能方法中的一种。

```
const model = {
  geometry: {
    url: 'geometry.json'
  },
  material: {
    map: 'map.jpg',
    normalMap: 'normalMap.jpg',
    metalness: 0.0,
    roughness: 1.0
  }
};loadMesh(model).then(mesh => {
  scene.add(mesh);
});function loadMesh(model) {
  const promises = [
    loadGeometry(model.geometry),
    loadMaterial(model.material)
  ];

  return Promise.all(promises).then(result => {
    return new THREE.Mesh(result[0], result[1]);
  });
}

function loadGeometry(model) {
  return new Promise(resolve => {
    new THREE.JSONLoader().load(model.url, resolve);
  });
}

const textureKeys = ['map', 'normalMap']; // etc...

function loadMaterial(model) {
  const params = {};
  const promises = Object.keys(model).map(key => {
    // load textures for supported keys
    if (textureKeys.indexOf(key) !== -1) {
      return loadTexture(model[key]).then(texture => {
        params[key] = texture;
      });
    // just copy the value otherwise  
    } else {
      params[key] = model[key];
    }
  });

  return Promise.all(promises).then(() => {
    return new THREE.MeshStandardMaterial(params);
  });
}
```

`model`描述网格的资产和任何附加设置。这是一个有用的抽象。我们可以创建许多模型，并使用上面的代码来创建相应的网格。如果你需要同时管理多个型号，可以很容易地将`createMesh`功能放到另一个`Promise.all()`中。从头到尾都是承诺。

```
const promises = models.map(model => {
  return loadMesh(model).then(mesh => {
    scene.add(mesh);
  });
});Promise.all(promises).then(() => {
  // load complete! begin rendering
});
```

这种方法的另一个好处是，可以用现有的或缓存的值来解决承诺，而无需对我们的 API 进行任何更改。这使得混合加载的和生成的资产变得容易。

```
const model = {
  geometry: {
    geometry: new THREE.SphereGeometry()
  },
  material: {...}
};

...

function loadGeometry(model) {
  if (model.geometry) {
    return Promise.resolve(model.geometry);
  }

  if (model.url) {
    return new Promise(resolve => {
      new JSONLoader().load(model.url, resolve);
    });  
  }
}
```

这些年来，我写了一些加载管理器，我可以告诉你，承诺大大简化了事情。您仍然需要添加自己的逻辑(缓存、错误处理、不同的加载器、附加参数等)，但是不必自己处理并发的异步请求可以节省大量时间。

这篇文章的片段可以在[这里](https://gist.github.com/zadvorsky/a79787a4703ecc74cab2fdbd05888e9b)找到。代码片段 06 包含了所有的最终函数，如果你粘贴它，应该可以工作。

我在[https://nike-react.com/](https://nike-react.com/)使用了一个类似的方法，这是迄今为止我做过的最重资产管理的项目。承诺，尤其是`Promise.all()`，有助于保持事情可控。