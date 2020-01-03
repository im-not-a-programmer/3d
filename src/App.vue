<template>
  <div id="mygoChart" style="width:100vw; height:100vh;"></div>
</template>

<script>
import go from "gojs";
export default {
  name: "app",
  mounted() {
    var mySelf = this;
    const MAKE = go.GraphObject.make;
    mySelf.myDiagram = MAKE(go.Diagram, "mygoChart", {
      initialContentAlignment: go.Spot.Center, // 居中显示
      "undoManager.isEnabled": false, // 支持 Ctrl-Z 和 Ctrl-Y 操作
      "toolManager.hoverDelay": 100, //tooltip提示显示延时
      "toolManager.toolTipDuration": 10000, //tooltip持续显示时间
      isReadOnly: true, //只读
      grid: MAKE(
        go.Panel,
        "Grid",
        { gridCellSize: new go.Size(20, 20) },
        MAKE(go.Shape, "LineH", { stroke: "lightgray" }),
        MAKE(go.Shape, "LineV", { stroke: "lightgray" })
      ),
      allowMove: false, //允许拖动
      allowDelete: false, // 是否可删除节点
      allowCopy: false, // 是否可复制对象
      allowClipboard: false,
      "toolManager.mouseWheelBehavior": go.ToolManager.WheelZoom, //有鼠标滚轮事件放大和缩小，而不是向上和向下滚动
      layout: MAKE(go.TreeLayout, {
        angle: 0,
        layerSpacing: 100
      })
    });

    mySelf.myDiagram.addDiagramListener("ObjectSingleClicked", function(e) {
      console.log("单击");
    });

    mySelf.myDiagram.addDiagramListener("BackgroundSingleClicked", function(e) {
      console.log("点击空白网格");
    });

    mySelf.myDiagram.addDiagramListener("ClipboardPasted", function(e) {
      console.log("Pasted" + e.diagram.selection.count + "parts");
    });

    // define a simple Node template
    // 定义个简单的 Node 模板
    mySelf.myDiagram.nodeTemplate = MAKE(
      go.Node,
      new go.Binding("location", "loc", go.Point.parse),
      MAKE(go.Shape, "RoundedRectangle", {
        fill: "#fff",
        stroke: "#BBBEC9",
        strokeWidth: 2,
        angle: 0
      }),
      "Auto", //Vertical,Auto,Horizontal
      // Node背景设置
      // { background: "#44CCFF" },

      // MAKE(go.Picture,
      //   // Pictures 应该指定宽高.
      //   // 当没有图片时显示红色的背景
      //   // 或者当图片为透明的时候也是.
      //   { source:"../assets/img/01.png",margin: 10, width: 50, height: 50, background: "red" },
      //   // Picture.source参数值与模型数据中的"source"字段绑定
      //   new go.Binding("source")),
      // MAKE(go.TextBlock,
      //   "Default Text",  // 初始化默认文本
      //   // 文字周围的空隙, 大号字体, 白色笔画:
      //   { margin: 12, stroke: "white", font: "bold 16px sans-serif",
      //     width:75,
      //     wrap: go.TextBlock.WrapDesiredSize
      //   },
      //   // TextBlock.text参数值与模型数据中的"name"字段绑定
      //   new go.Binding("text", "name1")),
      MAKE(
        go.Panel,
        "Horizontal",
        { padding: 5 },
        MAKE(
          go.Panel,
          "Vertical",
          MAKE(
            go.Picture,
            { margin: 10, width: 20, height: 20, background: "#fff" },
            new go.Binding("source", "img")
          )
        ),
        MAKE(
          go.Panel,
          "Vertical",
          MAKE(
            go.TextBlock,
            "Default Text",
            { margin: 12, stroke: "#40424C", font: "bold 16px sans-serif" },
            new go.Binding("text", "title")
          )
        )
      ),
      {
        mouseEnter: function(e, node, prev) {
          console.log("mouseEnter");
        },
        mouseLeave: function(e, node, prev) {
          mySelf.detailShow = false;
        }
      },
      {
        toolTip: MAKE(
          go.Adornment,
          "Spot",
          MAKE(go.Shape, "RoundedRectangle", {
            height: 30,
            fill: MAKE(go.Brush, "Linear", {
              1: "rgba(0, 0, 0, .8)",
              start: go.Spot.Bottom,
              end: go.Spot.Top
            })
          }),
          MAKE(
            go.TextBlock,
            //{alignment:go.Spot.Top,alignmentFocus:go.Spot.Bottom,stroke:"red" },
            { margin: 4, stroke: "white" },
            new go.Binding("text", "title")
          )
        ) // end of Adornment
      }
    );
    mySelf.myDiagram.linkTemplate = MAKE(
      go.Link,
      //{ curve: go.Link.Bezier },  // 贝塞尔曲线
      { routing: go.Link.Orthogonal, corner: 5 },
      MAKE(go.Shape, { strokeWidth: 2, stroke: "#BBBEC9" }),
      MAKE(go.Shape, { toArrow: "Standard", fill: "#BBBEC9", stroke: null }), //箭头
      // 箭头线 显示文字
      // MAKE(
      //   go.TextBlock,
      //   {
      //     margin: 20,
      //     stroke: "blue"
      //     font: "14px sans-serif",
      //     width:50,
      //     wrap: go.TextBlock.WrapDesiredSize
      //   },
      //   new go.Binding("text", "linktext")
      // ),
      {
        toolTip: MAKE(
          go.Adornment,
          "Auto",
          MAKE(go.Shape, { fill: "#FFFFCC" }),
          MAKE(go.TextBlock, { margin: 4 }, new go.Binding("text", "title"))
        ) // end of Adornment
      }
    ); // the link shape
    // let myModel = MAKE(go.Model);//如果不需要连线可以用这样的方法创建model
    // let myModel = MAKE(go.GraphLinksModel);//也可以创建link model;需要配置myModel.linkDataArray 如下
    let myModel = MAKE(go.TreeModel);
    myModel.nodeDataArray = [
      {
        key: "1",
        title: "流程配置1",
        name2: "秘书1"
      },
      {
        key: "2",
        parent: "1",
        title: "流程配置2",
        name2: "秘书1"
      },
      {
        key: "3",
        parent: "1",
        title: "流程配置3",
        name2: "秘书1",
        name3: "秘书2"
      },
      {
        key: "4",
        parent: "3",
        title: "流程配置4",
        name2: "秘书1",
        name3: "秘书2"
      },
      {
        key: "5",
        parent: "4",
        title: "流程配置5",
        name2: "秘书1"
      }
    ];
    // myModel.linkDataArray = [
    //   {from:"1",to:"2"},
    //   {from:"1",to:"3"},
    //   {from:"1",to:"4"},
    //   {from:"1",to:"5"},
    // ];
    // function diagramInfo(myModel) {
    //   return "myModel:\n" + myModel.nodeDataArray.length + " nodes, " +myModel.linkDataArray.length + " links";
    // }
    // mySelf.myDiagram.toolTip = MAKE(go.Adornment, "Auto",
    //   MAKE(go.Shape, { fill: "#CCFFCC" }),
    //   MAKE(go.TextBlock, { margin: 4 },
    //     // use a converter to display information about the diagram model
    //     new go.Binding("text", "", diagramInfo))
    // );
    mySelf.myDiagram.model = myModel;
  }
};
</script>
<style>
body {
  padding: 0;
  margin: 0;
}
</style>