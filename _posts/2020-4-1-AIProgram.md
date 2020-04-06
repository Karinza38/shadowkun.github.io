---                                                                                                                                                                                              
  layout: post

  title: AI Programming
---

## 智能化：人机协同的编程方式


> 在imgcook.com发布之后，我们在掘金、知乎、微信朋友圈和公众号发表了很多文章，收获最多的反馈是：前端要失业了。按照现在的趋势来看，对P5左右前端工程师的需求将大幅度降低。imgcook.com 官网底部有统计，从数据分析智能生成UI和逻辑代码的情况，imgcook 已经可以替代P5前端工程师完成工作的80%。无论趋势还是数据，都能看到传统前端低技术价值、高重复的工作正走向消亡。
>
> ![](https://cdn.nlark.com/yuque/0/2020/jpeg/164174/1586189804036-395fc9d4-369f-41f9-88d4-16ae9656e998.jpeg?x-oss-process=image/resize,w_1370)
>
> ​		如马云先生在香港对年轻人传授创业经验时讲到的，蒸汽机和电力解放了人类的体力，人工智能和机器学习解放了人类的脑力。马云先生在评价蒸汽机和电力带来的失业问题时讲到，人类在科技进步下从繁重的体力劳动中解放出来，逐步向脑力劳动过渡，这是人类社会的进步。今天前端从重复劳动的施工队里解放出来，逐步向智能化的高技术含量工作过渡，这是前端技术的进步。
> ​		综上所述，今天已经无法绕过智能化命题了，就像当今社会回不去刀耕火种的年代。预见到未来如此来临，我们推出了 github.com/alibaba/pipcook 项目，试图帮助前端零成本进入智能化，联合Google的Tensorflow.js团队，用这个框架轻松打通智能化的任督二脉：连接前端技术生态、复用Python技术生态。因此，后文将在Pipcook框架的基础上，从概念到方法、从方法到实践、从工具到应用的方式，和大家分享：前端的那些工作会被智能化取代？如何在工作中应用这种智能化方法？前端工程师如何借助 pipcook 和 AI 协同工作？最后，用imgcook详细设计实现作为案例，对实践进行系统的讲解。

除了去 imgcook.com 体验，这里带来一个 tensorflow.js 官方的示例，来看看智能化离我们的距离。首先是定义问题：很多人都很困惑发动机马力和行驶里程的关系是怎样的？是否可以根据发动机马力预测未来的这个车是否省油呢？传统统计分析的方法是定义模式，用模型计算分析，人和计算机是下指令和执行的主仆关系。机器学习的方法是通过数据学习模式，用模型学到的模式进行分析，人和计算机是提供数据和理解数据的协同关系。下面用代码展示这种协同怎么在Javascript里实现：

- 环境准备：

  - index.html

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <title>TensorFlow.js Tutorial</title>
    
      <!-- Import TensorFlow.js -->
      <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script>
      <!-- Import tfjs-vis -->
      <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-vis@1.0.2/dist/tfjs-vis.umd.min.js"></script>
    
      <!-- Import the main script file -->
      <script src="script.js"></script>
    
    </head>
    
    <body>
    </body>
    </html>
    ```

    这里Tensorflow.js就是谷歌官方的前端机器学习基础库，tfjs-vis是可视化工具用于显示数据、模型等信息，script.js就是示例的代码。

  - script.js

    示例的代码主要由四个部分构成：提供数据、定义模型、训练模型、模型预测，此外就是执行逻辑的 run 函数。下面针对这四个部分详细介绍一下。

- 提供数据：

  提供数据分为：获取数据、分析数据、处理数据三个阶段。获取数据可以理解为从现有的日志、云对象存储等数据源获取原始数据。分析数据则是对数据进行观察，寻找数据的问题：平均么？平凡么？符合现实场景么？对于我们定义的问题来说描述充分么？处理数据是根据分析的结果对数据进行校准，然后针对模型的训练要求进行格式化。

  - 获取数据：

    ```javascript
    async function getData() {
      const carsDataReq = await fetch(
        "https://storage.googleapis.com/tfjs-tutorials/carsData.json"
      );
      const carsData = await carsDataReq.json();
      return carsData;
    }
    ```

  - 分析数据：

    ```javascript
    async function run() {
      // Load and plot the original input data that we are going to train on.
      const data = await getData();
      const values = data.map((d) => ({
        x: d.horsepower,
        y: d.mpg,
      }));
    
      tfvis.render.scatterplot(
        { name: "Horsepower v MPG" },
        { values },
        {
          xLabel: "Horsepower",
          yLabel: "MPG",
          height: 300,
        }
      );
    }
    
    document.addEventListener("DOMContentLoaded", run);
    ```

  ![](https://cdn.nlark.com/yuque/0/2020/jpeg/164174/1586189823564-38b4071c-2d75-4744-bab2-7e0e498179db.jpeg)

  

  - 处理数据：

    洗菜：先对原始数据做一些清洗：

    ```javascript
    function cleaned(data){
      const cleaned = carsData
          .map((car) => ({
            mpg: car.Miles_per_Gallon,
            horsepower: car.Horsepower,
          }))
          .filter((car) => car.mpg != null && car.horsepower != null);
    
      return cleaned;
    }
    ```

    炒菜：再把数据加工成模型可以吃的样子：

    ```javascript
    function convertToTensor(data) {
      // Wrapping these calculations in a tidy will dispose any
      // intermediate tensors.
    
      return tf.tidy(() => {
        // Step 1. Shuffle the data
        tf.util.shuffle(data);
    
        // Step 2. Convert data to Tensor
        const inputs = data.map((d) => d.horsepower);
        const labels = data.map((d) => d.mpg);
    
        const inputTensor = tf.tensor2d(inputs, [inputs.length, 1]);
        const labelTensor = tf.tensor2d(labels, [labels.length, 1]);
    
        //Step 3. Normalize the data to the range 0 - 1 using min-max scaling
        const inputMax = inputTensor.max();
        const inputMin = inputTensor.min();
        const labelMax = labelTensor.max();
        const labelMin = labelTensor.min();
    
        const normalizedInputs = inputTensor
          .sub(inputMin)
          .div(inputMax.sub(inputMin));
        const normalizedLabels = labelTensor
          .sub(labelMin)
          .div(labelMax.sub(labelMin));
    
        return {
          inputs: normalizedInputs,
          labels: normalizedLabels,
          // Return the min/max bounds so we can use them later.
          inputMax,
          inputMin,
          labelMax,
          labelMin,
        };
      });
    }
    ```

    对于Tensor（张量）这个词不用在意，只要理解为像 Json 一样的数据组织格式就好了。这里唯一需要理解的就是“归一化”，因为数据可能有各种取值范围，如何把数据统一到一个取值范围内？上述代码把数据在 0 - 1 的范围内，借助极大值和极小值的约束对数据缩放，保证数据的本质并没发生改变只是表达方式变了。

    另一个概念就是Label（标签），这个很好理解，每种马力就是一个标签，数据是每加仑行驶的里程，示例加载的数据就是一个个每加仑行驶里程的数据和这条数据对应的发动机马力标签。

    ```
    ...
    {
      "mpg":15, //数据：每加仑行驶的里程
      "horsepower":165, //标签：发动机马力
    },
    {
      "mpg":18, //数据：每加仑行驶的里程
      "horsepower":150, //标签：发动机马力
    },
    {
      "mpg":16, //数据：每加仑行驶的里程
      "horsepower":150, //标签：发动机马力
    },
    ...
    ```

    用这样标注好的样本数据：菜，洗菜、炒菜之后喂给模型，模型吃了就能消化理解数据背后的模式和规律。

- 定义模型

  其实，除了研究机构和资深的学者外，真正能定义模型的人很少，算法工程师经历调参工程师、数据工程师后，仍然离定义模型很远，大部分时间连算法工程师都只是在应用而已。因此，去用就好了，定义模型只需要按照行业最流行、效果最好的模型抄作业即可。

  ```javascript
  /**
   * Define Model
   */
  function createModel() {
    // Create a sequential model
    const model = tf.sequential();
  
    // Add a single input layer
    model.add(tf.layers.dense({ inputShape: [1], units: 1, useBias: true }));
    model.add(tf.layers.dense({ units: 50, activation: "sigmoid" }));
    // Add an output layer
    model.add(tf.layers.dense({ units: 1, useBias: true }));
  
    return model;
  }
  ```

  如果你使用 github.com/alibaba/pipcook 连抄都省了，因为常用的模型我们和社区生态会提前抄好（移植好）。如果你需要抄一个，很简单，这里 tf 就是 tensorflow 的官方库名，函数定义和参数定义和 Python 里是一致的（语言特性造成的书写差异还需要具体修改），只要把 Tensorflow 官网提供的模型里的定义抄过来就好了。

  ![](https://cdn.nlark.com/yuque/0/2020/jpeg/164174/1586189845484-08ada28c-4358-4c0c-b3dc-ef25d5b838b5.jpeg)

- 训练模型

  不同于人和人之间的协作，机器学习的模型在训练之前是个小白，吃了我们炒的菜：标注数据 后，模型才有可能学会这些数据背后的模式和规律：

  ```javascript
  async function trainModel(model, inputs, labels) {
    // Prepare the model for training.
    model.compile({
      optimizer: tf.train.adam(),
      loss: tf.losses.meanSquaredError,
      metrics: ["mse"],
    });
  
    const batchSize = 32;
    const epochs = 50;
  
    return await model.fit(inputs, labels, {
      batchSize,
      epochs,
      shuffle: true,
      callbacks: tfvis.show.fitCallbacks(
        { name: "Training Performance" },
        ["loss", "mse"],
        { height: 200, callbacks: ["onEpochEnd"] }
      ),
    });
  }
  ```

  你没看错 model.fit 真的是喂 ta 唷，这里的 batchSize 和 epochs 就是超参数，而前文提到算法工程师是调参工程师就源于此， 他们大部分时间就真的是在调整这两个参数。这两个参数的意思很简单，batchSize 告诉模型一口吃多少？epochs告诉模型一次吃几口？老的模型一般怕吃多了噎着，或者不同任务下吃法不对，就像为了下饭可能会一口饭一口菜的吃，为了记住一种美味可能会连吃几口……

  ![](https://cdn.nlark.com/yuque/0/2020/jpeg/164174/1586189858549-65e36f2a-602d-497b-a867-08ad0ffb241f.jpeg)



- 模型预测

  吃完了标注数据这道菜，只要吃法正确，模型就能够记住这些数据背后的模式和规律，下一步自然是拉模型跟我们协作了：

  ```javascript
  function testModel(model, inputData, normalizationData) {
    const { inputMax, inputMin, labelMin, labelMax } = normalizationData;
  
    // Generate predictions for a uniform range of numbers between 0 and 1;
    // We un-normalize the data by doing the inverse of the min-max scaling
    // that we did earlier.
    const [xs, preds] = tf.tidy(() => {
      const xs = tf.linspace(0, 1, 100);
      const preds = model.predict(xs.reshape([100, 1]));
  
      const unNormXs = xs.mul(inputMax.sub(inputMin)).add(inputMin);
  
      const unNormPreds = preds.mul(labelMax.sub(labelMin)).add(labelMin);
  
      // Un-normalize the data
      return [unNormXs.dataSync(), unNormPreds.dataSync()];
    });
  ```

  这里的 model.predict 就是让模型对 tf.linspace 生成的随机数据进行预测，输出就是这数据大概是多少马力的发动机？因此，告诉模型每加仑行驶里程的数据，模型就能告诉我们这大概是多少马力的发动机，哒哒！协作完成。

  ![](https://cdn.nlark.com/yuque/0/2020/jpeg/164174/1586189871117-09f152ab-ae0e-43fa-9df0-531e0eef6f8e.jpeg)



- 完整代码

  ```javascript
  /**
   * Get the car data reduced to just the variables we are interested
   * and cleaned of missing data.
   */
  async function getData() {
    const carsDataReq = await fetch(
      "https://storage.googleapis.com/tfjs-tutorials/carsData.json"
    );
    const carsData = await carsDataReq.json();
    const cleaned = carsData
      .map((car) => ({
        mpg: car.Miles_per_Gallon,
        horsepower: car.Horsepower,
      }))
      .filter((car) => car.mpg != null && car.horsepower != null);
  
    return cleaned;
  }
  
  /**
   * Define Model
   */
  function createModel() {
    // Create a sequential model
    const model = tf.sequential();
  
    // Add a single input layer
    model.add(tf.layers.dense({ inputShape: [1], units: 1, useBias: true }));
    model.add(tf.layers.dense({ units: 50, activation: "sigmoid" }));
    // Add an output layer
    model.add(tf.layers.dense({ units: 1, useBias: true }));
  
    return model;
  }
  
  /**
   * Convert the input data to tensors that we can use for machine
   * learning. We will also do the important best practices of _shuffling_
   * the data and _normalizing_ the data
   * MPG on the y-axis.
   */
  function convertToTensor(data) {
    // Wrapping these calculations in a tidy will dispose any
    // intermediate tensors.
  
    return tf.tidy(() => {
      // Step 1. Shuffle the data
      tf.util.shuffle(data);
  
      // Step 2. Convert data to Tensor
      const inputs = data.map((d) => d.horsepower);
      const labels = data.map((d) => d.mpg);
  
      const inputTensor = tf.tensor2d(inputs, [inputs.length, 1]);
      const labelTensor = tf.tensor2d(labels, [labels.length, 1]);
  
      //Step 3. Normalize the data to the range 0 - 1 using min-max scaling
      const inputMax = inputTensor.max();
      const inputMin = inputTensor.min();
      const labelMax = labelTensor.max();
      const labelMin = labelTensor.min();
  
      const normalizedInputs = inputTensor
        .sub(inputMin)
        .div(inputMax.sub(inputMin));
      const normalizedLabels = labelTensor
        .sub(labelMin)
        .div(labelMax.sub(labelMin));
  
      return {
        inputs: normalizedInputs,
        labels: normalizedLabels,
        // Return the min/max bounds so we can use them later.
        inputMax,
        inputMin,
        labelMax,
        labelMin,
      };
    });
  }
  
  async function trainModel(model, inputs, labels) {
    // Prepare the model for training.
    model.compile({
      optimizer: tf.train.adam(),
      loss: tf.losses.meanSquaredError,
      metrics: ["mse"],
    });
  
    const batchSize = 32;
    const epochs = 50;
  
    return await model.fit(inputs, labels, {
      batchSize,
      epochs,
      shuffle: true,
      callbacks: tfvis.show.fitCallbacks(
        { name: "Training Performance" },
        ["loss", "mse"],
        { height: 200, callbacks: ["onEpochEnd"] }
      ),
    });
  }
  
  function testModel(model, inputData, normalizationData) {
    const { inputMax, inputMin, labelMin, labelMax } = normalizationData;
  
    // Generate predictions for a uniform range of numbers between 0 and 1;
    // We un-normalize the data by doing the inverse of the min-max scaling
    // that we did earlier.
    const [xs, preds] = tf.tidy(() => {
      const xs = tf.linspace(0, 1, 100);
      const preds = model.predict(xs.reshape([100, 1]));
  
      const unNormXs = xs.mul(inputMax.sub(inputMin)).add(inputMin);
  
      const unNormPreds = preds.mul(labelMax.sub(labelMin)).add(labelMin);
  
      // Un-normalize the data
      return [unNormXs.dataSync(), unNormPreds.dataSync()];
    });
  
    const predictedPoints = Array.from(xs).map((val, i) => {
      return { x: val, y: preds[i] };
    });
  
    const originalPoints = inputData.map((d) => ({
      x: d.horsepower,
      y: d.mpg,
    }));
  
    tfvis.render.scatterplot(
      { name: "Model Predictions vs Original Data" },
      {
        values: [originalPoints, predictedPoints],
        series: ["original", "predicted"],
      },
      {
        xLabel: "Horsepower",
        yLabel: "MPG",
        height: 300,
      }
    );
  }
  
  async function run() {
    // Load and plot the original input data that we are going to train on.
    const data = await getData();
    const values = data.map((d) => ({
      x: d.horsepower,
      y: d.mpg,
    }));
  
    tfvis.render.scatterplot(
      { name: "Horsepower v MPG" },
      { values },
      {
        xLabel: "Horsepower",
        yLabel: "MPG",
        height: 300,
      }
    );
  
    // More code will be added below
    // Create the model
    const model = createModel();
    tfvis.show.modelSummary({ name: "Model Summary" }, model);
  
    // Convert the data to a form we can use for training.
    const tensorData = convertToTensor(data);
    const { inputs, labels } = tensorData;
  
    // Train the model
    await trainModel(model, inputs, labels);
    console.log("Done Training");
  
    // Make some predictions using the model and compare them to the
    // original data
    testModel(model, data, tensorData);
  }
  
  document.addEventListener("DOMContentLoaded", run);
  ```

  这就完成了整个买菜（获取数据）、洗菜（处理数据）、炒菜（处理数据）、猎头（寻找/抄/定义模型）、投食（训练模型）、协作（模型预测）的过程，全部都是在浏览器内完成（Tensorflow.js 已经加入了WASM的加速能力，未来会支持 WebNN 的加速能力），全部都是前端技术，你说前端智能化是否已经到来？

- What`s next

  - 还在发布上线的时候盯屏看各种指标？把 日志和指标 组织成示例中 每加仑行驶里程和发动机马力 的格式，拿上面的代码训练个模型，写个Chrome插件来盯屏吧。
  - 把本书看完，系统学习智能化，在前端领域进行人机协同的编程。
  - 用本书介绍的原理和方法，去 https://www.tensorflow.org/resources/models-datasets 抄更多的模型。

## 背景

### 历史

在此先回顾一下前端技术发展的脉络，以期从脉络中为大家整理出技术发展的规律，从而引出智能化的发端。HTML和CSS是第一阶段，超文本协议下承载图文内容，是前端技术最初的样子。生产过程大致是网页三剑客：Dreamweaver、Fireworks、Flash，Fireworks用来切图，Dreamweaver用来以所见即所得的方式编辑HTML和CSS，Flash用来实现富媒体体验如：音频、动画、视频等。这时网页还是静态的，在设计和实现的过程中决定了最终展示给用户的内容和样式。

引入脚本能力和DHTML是第二阶段，DHTML带来了DOM（文档对象模型）体系，针对ECMA标准扩展Javascript语言的浏览器厂商，带来了BOM（浏览器对象模型）体系，脚本带来了逻辑控制和数据处理能力，他们共同成就了网页动态化能力。所谓动态化能力是指，在网页设计和实现之后，在运行时脚本和BOM结合对DOM进行操作，对网页内容和样式进行改变。生产过程除了Dreamweaver里具备了脚本编辑、调试、代码段儿管理，可基本完成动态页面开发工作，还有比较专业的IDE出现。浏览器端脚本也从VBScript、JScript、Javascript统一到Javascript，Flash长出了Action Script和Flex技术，微软也提出Silverlight技术把 .Net 语言引入，后来加入ECMA标准的XPath广义上说也可以理解为脚本语言，用XSLT格式化XML实现页面的渲染和控制中，XPath起到了关键作用。

HTTPWebRequest推出在BOM领域对前端技术带来巨大影响，直接催生了锋利的JQuery框架。在维持原有的动态化能力外，统一收敛脚本的能力，用面向研发的视角把框架能力抽象为：网络能力、数据能力、DOM选择能力、DOM操作能力……等。生产过程从原始的文件组织，升级到各种脚手架初始化和辅助开发，从Ctrl+C和Ctrl+V的代码片段式升级到Library和Module方式，增加了DOM和BOM技术易用性，也增加了代码的复用性、可读性和可维护性，因此，前端技术加持下出现了更加复杂的Single Page Application和Web Application。

随着Library和Module无法满足业务诉求，Component用更加细粒度的抽象提供了更丰富的业务表现力。生产过程也从直接开发、现场施工进化到间接开发、预制施工，完成产品的工序从研发移动到搭建。因为需要从业务角度进行抽象，所以这种生产方式适合成熟产品和领域，否则，难有足够可用的Component实现产品。对于新产品和新业务，没有足够的代码、组件、模块、库、框架、脚手架的沉淀。因此，面对新产品和新业务的生产过程，有大量的新组件、新模块开发工作，甚至可能要修改库、框架和脚手架以提供新的业务表现力。因此，再丰富的组件体系在面对复杂多变的业务需求，仍然显得力不从心。

最初，前端工程师跟HTML协同形成Web雏形，然后跟DHTML、Javascript等技术协同形成动态Web内容，最后在HTTPWebRequest这个BOM对象协助下产生Web 2.0，“智能化”将标志着前端工程师和机器协同的新时代来临。

### 智能化发端

 2003年7月毕业于中央司法警官学院，辗转4年后2007年6月进入腾讯，在前端开发通道三年半从T1.1升到T3.3，切实为我打开了广阔的天地。走到今天一直对前端行业为我带来的价值心存感恩，看到智能化将是前端工程师和机器协同新时代的契机，我毅然决然肩负起设计、实现、布道前端智能化的责任，以期反哺前端行业，在前端智能化进程中留下自己存在过的证明。

先是尝试在营销活动领域，尝试用智能化手段对前端的搭建体系进行升级；后在前端自动化测试领域，引入机器学习和计算机视觉辅助UI测试。虽然都有所收获，但始终无法让我满意。究竟如何才能让“智能化”像HTTPWebRequest一样给前端带来质变？为此，我迫使自己回到十五年的软件研发经验中去反思：

- 第一个五年

首先，我考察自己做开发的第一个五年。每次接到需求时，大概率只是个粗放的Idea。这个Idea进入我脑海中后，我会搜索自己技术的"专家系统"，试图解答这个Idea的可行性。PD收到我的反馈后会调整Idea，PD在期望和反馈之间找到一个平衡点。然后，PD回去跟设计师PK，大概与跟我PK的过程相似。设计师会搜索自己的"专家系统"，试图解答Idea长什么样？支持哪些交互方式？视觉上变化过程和结果是什么？PD收到设计师的反馈后会调整预期，在预期和反馈之间找到一个平衡点。最后，PD把PK过程的产出物梳理一下，形成一套完整的PRD、交互稿、视觉稿，在通过项目评审之后，交给我进行开发。

第一个五年总结下来， 最大的启发是：研发就是把PRD、交互稿、视觉稿组成的需求翻译为程序的过程。开发过程中，先由PD讲解PRD，把PRD、交互稿、视觉稿缺失的信息通过讲解补全，再由我对所有细节对照技术实现手段进行概念的映射。比如：用户名输入、密码输入、登陆这三个产品概念，在技术实现上对应到文本输入、表单提交、鉴权、存储凭证等，把一系列产品概念翻译成技术实现上的概念，从而形成架构设计、概要设计、详细设计然后编码。这个生产过程就是程序员把PD的需求翻译成代码，智能化就是算法模型把PD的需求翻译成代码。本文第一部分将介绍智能化原理、原则、概念、方法，为后续的实践打下基础。

- 第二个五年

第二个五年里，我会在上述生产过程之外，加入横向能力的思考。技术上需要一纵一横，一纵指的是在一个产品、一个业务、一个领域持续的学习、实践和积累，一横指的是在对这个产品、业务领域进行提炼和抽象，把提炼和抽象的结果用技术产品化的方式横向复用到更多产品、业务和事情上。

第二个五年总结下来，最大的启发是：授人以鱼不如授人以渔。我需要把 imgcook.com 实现过程中创造并沉淀的能力分享出来，用 github.com/alibaba/pipcook 这个项目开源给行业。pipcook作为前端的机器学习框架，凭借原生Javascript开发能力打通现有NPM生态和Python的Pip生态，能够在更多的产品、业务上给前端创造发挥空间。本文的第二部分会详细介绍如何安装、使用和自定义pipcook，用案例和源码阐释智能化的易用和高效。

- 第三个五年

最近五年里，我会在上述生产过程之外，加入全局视角。所谓全局视角犹如阿里前端委员会主席圆心在GMTC上分享的"双视角"一样，同时用业务视角和技术视角去看待生产过程。业务视角下技术能带来什么价值？技术视角下如何易用、高效支撑业务？

第三个五年总结下来，最大的启发是：商业是技术的原动力，交付业务是技术的终极目标。本文最后会在 imgcook 的智能化实践案例中，详细分享如何用pipcook作为基础来实现完整的智能化代码生成和业务交付。

## 原理篇

> 在我看来,看清**AI 能做什么不能做什么,将目标聚焦在可以*100%*控制的,能有效提升我们生产力与行动力的成果上,承认只有"人*+*机器"的组合才是*AI*研究的主流方向,这或许更有意义,也是人类社会发展的正确方向。**
>
> ——微软亚洲研究院院长 [洪小文](http://research.microsoft.com/en-us/people/hon/)

### 智能化现状

下面先从现状进行分析，之后再介绍过程中的核心智能化概念和方法。

我们落地这个解题思路的核心产品是imgcook.com，imgcook.com的核心能力是D2C（Design to Code），D2C能力的核心是模仿前端工程师完成页面生产过程。这个探索和实现的过程经历了三个阶段：R2C、D2C、P2C。

- 第一阶段：R2C

R2C指Rule to code也就是规则生成代码，借助设计稿约束规则和设计稿源文件分析规则结合，通过设计稿生成代码。当2018年9月来到杭州时，imgcook.com的前身“达芬奇”已经做了两年多了，为了配套达芬奇的设计稿生成代码能力，底层构建了数据标准化和统一投放体系，上层构建了方舟、命运石等业务平台，但能够解决的问题非常有限，只有严格遵守设计稿规范且比较简单的问题才能被“达芬奇”解决。深入分析，达芬奇依赖OpenCV技术对设计稿进行识别，OpenCV里需要大量的阈值约束才能达到比较好的效果。事实上，无论我们在OpenCV里怎么调整阈值，总是在各种Badcase中间闪转腾挪却不得其门而入，在设计稿无限的可能性和无限的设计师个人习惯之无限组合下，规则的复杂度爆炸，R2C无法实现替代程序员的目标。

- 第二阶段：D2C

D2C指Design to code也就是设计稿生成代码，借助机器学习和计算机视觉对设计稿进行识别和理解，结合图像元素特征和上下文特征等信息，模型推理出元素是什么。这个过程用于解决R2C无法解决的规则爆炸问题，遇到无法确定规则的地方由模型进行识别。这个生产过程像极了前端的页面重构岗位，看设计稿、切图、编写UI和交互代码。2018年9月到2019年1月D2前端论坛宣布开放 imgcook.com，我用近四个月的时间和达芬奇团队妙净、波本、卡狸、芸明等同学一起，完成了OpenCV阈值部分由模型替代的工作，在去规则上取得突破性的成果。在第二阶段，imgcook已经初步实现了智能化和编程协同生成代码的目标，借助IDE插件、imgcook studio、Open CLI、Open DSL和模型服务等能力，前端工程师和前端智能化模型相互协同，在大部分工作场景里实现了约40%左右的效率提升。

- 第三阶段：P2C

P2C指Productive to code也就是产品化代码生成，借助需求结构化收集、需求理解、业务逻辑和代码逻辑片段理解、数据字段理解、API理解，完成从需求理解到代码生成，能够高效完整的交付业务。这个生产过程超越了页面重构，imgcook 更加接近一个P5前端工程师的水平。在页面重构到P5前端工程师的差距很大，显然无法一蹴而就。我们设计了一个三步走的路径：简单模块开发、复杂多态和交互模块开发、完整产品交付，从生产方式上可以划分为两个类型：生产模块然后搭建，生产产品然后交付。

第一步：简单模块开发（生产模块然后搭建）

何谓简单模块？我们定义为在双十一上使用，且没有强交互和复杂状态变化的模块。目标是双十一简单模块零研发，方案是在imgcook.com基础上，协同方舟（大促的搭建和投放系统）的同学一起共建，用D2C能力供给模块、用方舟搭建投放消费模块，实现了imgcook和方舟之间的协同。

为了解决“生成逻辑代码”这个最棘手的问题，项目开始前我在会议中提出：在存量代码的基础上，如果能提炼和抽象出通用逻辑，再借助模型识别和推理确定使用哪些逻辑，我们就能够解决逻辑代码生成的问题，保证模块开发后能够在方舟上搭建使用。为了自动化“数据绑定”，在模块语义化和数据字段理解与匹配上引入了智能化能力。

最终，通过上述方法的全新智能生产过程，我们在数据字段自动化绑定获得了30%的准确率、逻辑点识别和逻辑代码生成准确率超过60%、整体D2C代码可用率80%（生成代码后未被修改的比例），在两个月的时间里和方舟团队一起完成了双十一简单模块零研发的目标。

第二步：复杂多态和交互模块开发（生产模块然后搭建）

在复杂多态和交互模块开发方面，智能生成遇到了D2C以来最大的困难：信息缺失。一个P5程序员能够完成开发，如前所述依赖于：PD讲解PRD，把PRD、交互稿、视觉稿缺失的信息通过讲解补全。那么，PRD、交互稿、视觉稿就是智能生成的基本条件。从现实情况来看，大部分PD不太能完整产出PRD、交互稿、视觉稿，同时，不同的习惯和不同的工具造成不同的产出物，增加了对PRD、交互稿、视觉稿理解的成本。视觉稿因为由图像领域工业化程度很高的模型能力决定，可以不做太多约束，从PD的视角看，今天C端业务基本都会和设计师一起产出视觉稿，否则项目评审都无法通过。PRD和交互稿的质量却是参差不齐、格式也是千差万别，结构化理解难度极高。对于信息缺失，通过对PRD和交互稿进行结构化约束，可以弥补无法获取完整产品设计信息的漏洞。

因此，在第二步浩知带领团队设计实现了ideacook平台。在ideacook的设计上，首先从imgcook生成复杂多台和交互模块的问题出发，找到良好生成代码所需要输入的信息，然后通过imgcook把设计稿还原，再由PD可视化补全这些必要信息：when（用户点击）then（跳转）（目标路由地址）。

第三步：

### 智能化定义

打通功能、智能和智力、智慧的边界，以人机协同的方式进行软件开发
人机协同的编程方式

### 智能化概念

#### 功能(Capability)

#### 智能(Intelligence)

#### 智力(Intellect)

#### 智慧(Wisdom)

### 智能化特性

#### 确定性

#### 鲁棒性

#### 进化性

### 智能化原则

#### 面向数据原则

#### 实现之实现原则

#### 人机竞合原则

#### 反馈闭环原则

### 智能化模式

#### 决策模式

#### 创意模式

#### 先验模式

#### 后验模式

#### 创建模式

#### 结构模式

#### 行为模式
