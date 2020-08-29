<template>
  <div class="container">
    <h1 class="title">Flowchart Vue</h1>
    <!-- <h5 class="subtitle">
      Flowchart & Flowchart designer component for Vue.js.
    </h5> -->
    <div id="toolbar">
      <button
        @click="
          $refs.chart.add({
            id: +new Date(),
            x: 10,
            y: 10,
            thumbnail: 'https://placehold.jp/150x100.png',
            url: '',
            name: 'Title',
            summary: 'Summary',
          })
        "
      >
        Add(Double-click canvas)
      </button>
      <button @click="$refs.chart.remove()">Delete(Del)</button>
      <button @click="$refs.chart.editCurrent()">
        Edit(Double-click node)
      </button>
      <button @click="$refs.chart.save()">Save</button>
    </div>
    <flowchart
      :nodes="nodes"
      :connections="connections"
      @editnode="handleEditNode"
      :readonly="false"
      @dblclick="handleDblClick"
      @editconnection="handleEditConnection"
      @save="handleChartSave"
      ref="chart"
      :render="render"
    >
    </flowchart>
    <node-dialog
      :visible.sync="nodeDialogVisible"
      :node.sync="nodeForm.target"
    ></node-dialog>
    <connection-dialog
      :visible.sync="connectionDialogVisible"
      :connection.sync="connectionForm.target"
      :operation="connectionForm.operation"
    >
    </connection-dialog>
  </div>
</template>
<!-- <script src="//d3plus.org/js/d3.js"></script>
<script src="//d3plus.org/js/d3plus.js"></script> -->
<script>
/* eslint-disable no-unused-vars */

import ConnectionDialog from "../components/ConnectionDialog";
import NodeDialog from "../components/NodeDialog";
import Flowchart from "../components/flowchart/Flowchart";
import * as d3 from "d3";
import { roundTo20 } from "../utils/math";

export default {
  components: {
    ConnectionDialog,
    NodeDialog,
    Flowchart,
  },
  data: function () {
    return {
      nodes: [
        {
          id: 1,
          x: 200,
          y: 50,
          thumbnail: "https://qiita-user-contents.imgix.net/https%3A%2F%2Fcdn.qiita.com%2Fassets%2Fpublic%2Farticle-ogp-background-1150d8b18a7c15795b701a55ae908f94.png?ixlib=rb-1.2.2&w=1200&mark=https%3A%2F%2Fqiita-user-contents.imgix.net%2F~text%3Fixlib%3Drb-1.2.2%26w%3D840%26h%3D380%26txt%3DDocker%25E3%2581%25A8%25E3%2581%25AF%25E3%2581%25A9%25E3%2581%2586%25E3%2581%2584%25E3%2581%25A3%25E3%2581%259F%25E3%2582%2582%25E3%2581%25AE%25E3%2581%25AA%25E3%2581%25AE%25E3%2581%258B%25E3%2580%2581%25E3%2582%2581%25E3%2581%25A1%25E3%2582%2583%25E3%2581%258F%25E3%2581%25A1%25E3%2582%2583%25E4%25B8%2581%25E5%25AF%25A7%25E3%2581%25AB%25E8%25AA%25AC%25E6%2598%258E%25E3%2581%2597%25E3%2581%25A6%25E3%2581%25BF%25E3%2582%258B%26txt-color%3D%2523333%26txt-font%3DHiragino%2520Sans%2520W6%26txt-size%3D54%26txt-clip%3Dellipsis%26txt-align%3Dcenter%252Cmiddle%26s%3Da74a6782c81bee821aa421a1037a1cc0&mark-align=center%2Cmiddle&blend=https%3A%2F%2Fqiita-user-contents.imgix.net%2F~text%3Fixlib%3Drb-1.2.2%26w%3D840%26h%3D500%26txt%3D%2540SatoshiSobue%26txt-color%3D%2523333%26txt-font%3DHiragino%2520Sans%2520W6%26txt-size%3D45%26txt-align%3Dright%252Cbottom%26s%3Dca9d81597f6bcfd55d58a95dc4c5f445&blend-align=center%2Cmiddle&blend-mode=normal&s=6d406ff15d17893a52ab77ba3ebd9d33",
          url: 'https://qiita.com/SatoshiSobue/items/a612ebbb3a9242c09db5',
          name: "Dockerとはどういったものなのか、めちゃくちゃ丁寧に説明してみる - Qiita",
          summary: "仮想化などDockerの周辺知識も含めて丁寧に解説されている。",
        },
        {
          id: 2,
          x: 200,
          y: 300,
          thumbnail: "https://res.cloudinary.com/practicaldev/image/fetch/s--Zm43GYiy--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/e4jmzv9o1ivd3yz2kj8a.png",
          url: 'https://dev.to/mizuki04/tips-docker-5bfj',
          name: "[Tips] 中銀カプセルタワービル=Docker説 - DEV",
          summary: "Dockerの概要が面白い例えで紹介されている。イメージをつかめる。	",
        },
        // {
        //   id: 3,
        //   x: 340,
        //   y: 130,
        //   name: "Custom size",
        //   thumbnail: "operation",
        //   approvers: [{ id: 1, name: "Joyce" }],
        //   // width: 120,
        //   // height: 40,
        // },
        {
          id: 4,
          x: 50,
          y: 700,
          thumbnail: "https://img-a.udemycdn.com/course/480x270/3207931_c941.jpg",
          url: 'https://www.udemy.com/course/aidocker/',
          name: "米国AI開発者がゼロから教えるDocker講座",
          // approvers: [{ id: 2, name: "Allen" }],
          summary: "インプットしているだけではわからないので、手を動かして覚える。過去一わかりやすい動画。	",
        },
        {
          id: 5,
          x: 400,
          y: 700,
          thumbnail: "https://i.ytimg.com/vi/Fq1PH0Gwi8I/hqdefault.jpg",
          url: 'https://www.youtube.com/watch?v=Fq1PH0Gwi8I',
          name: "【rails環境構築】docker + rails + mysql で環境構築（初心者でも30分で完了！）",
          // approvers: [{ id: 3, name: "Teresa" }],
          summary: "無料がいい方はこちら。	",
        },
      ],
      connections: [
        {
          // positionは矢印がつながっているコネクターの位置っぽい
          source: { id: 1, position: "bottom" },
          destination: { id: 2, position: "top" },
          id: 1,
          type: "pass",
          name: "たとえ話でイメージをつかむ"
        },
        {
          source: { id: 2, position: "bottom" },
          destination: { id: 4, position: "top" },
          id: 2,
          type: "pass",
          name: "実際に手を動かす"
        },
        {
          source: { id: 2, position: "bottom" },
          destination: { id: 5, position: "top" },
          id: 3,
          type: "pass",
          name: "実際に手を動かす"
        },
        // {
        //   source: { id: 5, position: "bottom" },
        //   destination: { id: 4, position: "bottom" },
        //   id: 4,
        //   type: "reject",
        // },
        // {
        //   source: { id: 1, position: "top" },
        //   destination: { id: 3, position: "left" },
        //   id: 5,
        //   type: "pass",
        // },
        // {
        //   source: { id: 3, position: "right" },
        //   destination: { id: 2, position: "top" },
        //   id: 6,
        //   type: "pass",
        // },
      ],
      nodeForm: { target: null },
      connectionForm: { target: null, operation: null },
      nodeDialogVisible: false,
      connectionDialogVisible: false,
    };
  },
  async mounted() {},
  methods: {
    handleDblClick(position) {
      this.$refs.chart.add({
        id: +new Date(),
        x: 10,
        y: 10,
        thumbnail: 'https://placehold.jp/150x100.png',
        url: '',
        name: 'Title',
        summary: 'Summary',
      });
    },
    async handleChartSave(nodes, connections) {
      // axios.post(url, {nodes, connection}).then(resp => {
      //   this.nodes = resp.nodes;
      //   this.connections = resp.connections;
      //   // Flowchart will refresh after this.nodes and this.connections changed
      // });
    },
    handleEditNode(node) {
      this.nodeForm.target = node;
      this.nodeDialogVisible = true;
    },
    handleEditConnection(connection) {
      this.connectionForm.target = connection;
      this.connectionDialogVisible = true;
    },
    render: function (g, node, isSelected) {
      node.width = node.width || 400;
      node.height = node.height || 200;
      let borderColor = isSelected ? "#666666" : "#bbbbbb";

      // // バイト数カウント関数を定義
      // String.prototype.bytes = function () {
      //   return(encodeURIComponent(this).replace(/%../g,"x").length);
      // }

      // function leftLinebreak(array){
      //   let string = "";
      //   array.forEach((t, i) =>{
      //     string += `<tspan y="${i}em" x="0em">${t}</tspan>`;
      //     string += `<tspan y="${i}em" x="0em">${t}</tspan>`;
      //   });
      //   return string;
      // }

      // image
      g.append('foreignObject')
        .attr("x", node.x)
        .attr("y", node.y)
        .style("width", node.width * 3 / 8 + "px")
        .style("height", node.height / 2 + "px")
        .append('xhtml:div')
        .style("width", node.width * 3 / 8 + "px")
        .style("height", node.height / 2 + "px")
        // .attr("class", "imgaaa")
        // .style("background-image", "url('https://res.cloudinary.com/practicaldev/image/fetch/s--Zm43GYiy--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/e4jmzv9o1ivd3yz2kj8a.png')")
        .style("background-image", `url(${node.thumbnail})`)
        .style("background-position", "center center")
        .style("background-size", "cover")
        .style("background-repeat", "no-repeat")
        .style("border-radius", "6px 0 0 0")
        .style("box-sizing", "border-box")
        // .style("border-top", "1px solid #7CF8FD")
        // .style("border-left", "1px solid #7CF8FD")
        .style("border-top", "1px solid white")
        .style("border-left", "1px solid white")
        // .style("background-color", "red")
        // .append('img')
        // .attr('src', 'https://res.cloudinary.com/practicaldev/image/fetch/s--Zm43GYiy--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/e4jmzv9o1ivd3yz2kj8a.png')
        // .style("width", node.width / 2 + "px")
        // .style("height", node.height / 2 + "px")
        // .text(() => node.name)

      // g.append("image")
      //   .attr("x", node.x)
      //   .attr("y", node.y)
      //   .attr('xlink:href', 'https://res.cloudinary.com/practicaldev/image/fetch/s--Zm43GYiy--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/e4jmzv9o1ivd3yz2kj8a.png')
      //   .attr("preserveAspectRatio", "none")
      //   // .attr("preserveAspectRatio", "xMidYMid meet")
      //   .style("width", node.width / 2 + "px")
      //   .style("height", node.height / 2 + "px")

        //   .attr("stroke", borderColor)
        //   .attr("class", "title")
        //   .style("height", "20px")
        //   .style("fill", "#f1f3f4")
        //   .style("stroke-width", "1px")
        //   .style("width", node.width + "px");
        // g.append("text")
        //   .attr("x", node.x + 4)
        //   .attr("y", node.y + 15)
        //   .attr("class", "unselectable")
        //   .text(() => node.name)
        //   .each(function wrap() {
        //     let self = d3.select(this),
        //       textLength = self.node().getComputedTextLength(),
        //       text = self.text();
        //     while (textLength > node.width - 2 * 4 && text.length > 0) {
        //       text = text.slice(0, -1);
        //       self.text(text + "...");
        //       textLength = self.node().getComputedTextLength();
        //     }
        // });

      // title
      // g.append("rect")
      //   .attr("x", node.x + node.width * 3 / 8)
      //   .attr("y", node.y)
      //   .attr("stroke", borderColor)
      //   .attr("class", "title")
      //   .style("height", node.height / 2 + "px")
      //   .style("fill", "#f1f3f4")
      //   .style("fill", "#3F3F3F")
      //   .style("stroke-width", "1px")
      //   .style("width", node.width * 5 / 8 + "px")

      g.append('foreignObject')
        .attr("x", node.x + node.width * 3 / 8)
        .attr("y", node.y)
        .attr("class", "title")
        .style("width", node.width * 5 / 8 + "px")
        .style("height", node.height / 2 + "px")
        .append('xhtml:div')
        // .append('p')
        .style("width", node.width * 5 / 8 + "px")
        .style("height", node.height / 2 + "px")
        // .style("background-color", "red")
        .style("background-color", "#3F3F3F")
        .style("border-radius", "0 6px 0 0")
        .style("box-sizing", "border-box")
        // .style("border-top", "1px solid #7CF8FD")
        // .style("border-right", "1px solid #7CF8FD")
        .style("border-top", "1px solid white")
        .style("border-right", "1px solid white")

      g.append('foreignObject')
        .attr("x", node.x + node.width * 3 / 8)
        .attr("y", node.y)
        .attr("class", "unselectable")
        .style("width", node.width * 5 / 8 + "px")
        .style("height", node.height / 2 + "px")
        .style("display", "table")
        .append('xhtml:p')
        // .append('p')
        .style("display", "table-cell")
        .style("vertical-align", "middle")
        .style("width", node.width * 5 / 8 + "px")
        .style("height", node.height / 2 + "px")
        .style("box-sizing", "border-box")
        .style("padding", "4px 8px 4px 8px")
        .style("color", "white")
        .style("font-weight", "bold")
        .style("margin", 0)
        // .style("word-wrap", "break-word")
        .style("overflow-wrap", "break-word")
        // .style("color", "red")
        // .html('test test test   test test testtesttesttesttesttesttesttest')
        .text(() => node.name)

      // Wrap text in a rectangle.
      // d3plus.textwrap()
      //   .container(d3.select("#rectWrap"))
      //   .draw();

      // let textArray = []
      // let summary = g.append("text")
      //   .attr("x", node.x + node.width / 2 + 4)
      //   .attr("y", node.y + 20)
      //   .attr("class", "unselectable")
      //   .text(() => node.name)
      //   .each(function wrap() {
      //     let self = d3.select(this),
      //       textLength = self.node().getComputedTextLength(),
      //       text = self.text();
      //       console.log('bytes: ' + text.bytes());
      //       textArray = []
      //       let slicedText
      //     while (text.bytes() > 16 && text.length > 0) {
      //       slicedText = text.slice(0, 16);
      //       textArray.push(text.slice(0, 16));
      //       text = text.slice(16);
      //       console.log('textArray: ' + textArray);
      //       console.log('textArray.length: ' + textArray.length);
      //       console.log('text: ' + text);
      //       // self.text(text + "...");
      //       // textLength = self.node().getComputedTextLength();
      //     }
      //     // while (textLength > node.width / 2 - 2 * 4 && text.length > 0) {
      //     //   text = text.slice(0, -1);
      //     //   self.text(text + "...");
      //     //   textLength = self.node().getComputedTextLength();
      //     // }
      //   }
      //   ).html(leftLinebreak(textArray))
      //   // ).html('<foreignObject><p>テスト</p></foreignObject>')
      //   // summary.html(leftLinebreak(textArray))

      // body
      // if (node.id === 3) {
      //   let body = g.append("ellipse").attr("class", "body");
      //   body.attr("cx", node.x + node.width / 2);
      //   body.attr("cy", node.y + node.height / 2);
      //   body.attr("rx", node.width / 2);
      //   body.attr("ry", node.height / 2);
      //   body.style("fill", "white");
      //   body.style("stroke-width", "1px");
      //   body.classed(node.thumbnail, true);
      //   body.attr("stroke", borderColor);
      // } else {
      //   let body = g.append("rect").attr("class", "body");
      //   body
      //     .style("width", node.width + "px")
      //     .style("fill", "white")
      //     .style("stroke-width", "1px")
      //     .style("fill", "#3F3F3F")
      //   // if (node.type !== "start" && node.type !== "end") {
      //     body
      //       .attr("x", node.x)
      //       .attr("y", node.y + node.height / 2)
      //       // .style("height", roundTo20(node.height / 2  - 20) + "px");
      //       .style("height", node.height / 2 + "px")
      //   // } else {
      //     // body
      //     //   .attr("x", node.x)
      //     //   .attr("y", node.y)
      //     //   .classed(node.type, true)
      //     //   .attr("rx", 30);
      //     // body.style("height", roundTo20(node.height) + "px");
      //   // }
      //   body.attr("stroke", borderColor);
      // // }
      g.append('foreignObject')
        .attr("x", node.x)
        .attr("y", node.y + node.height / 2)
        // .attr("class", "body")
        .style("width", node.width + "px")
        .style("height", node.height / 2 + "px")
        .append('xhtml:div')
        // .append('p')
        .style("width", node.width + "px")
        .style("height", node.height / 2 + "px")
        // .style("margin", 0)
        .style("background-color", "#707070")
        // .style("border-radius", "0 4px 0 0")
        .style("box-sizing", "border-box")
        .style("border-radius", "0 0 6px 6px")
        // .style("border-top", "3px solid #6A6A6A")
        // .style("border-bottom", "1px solid #7CF8FD")
        // .style("border-right", "1px solid #7CF8FD")
        // .style("border-left", "1px solid #7CF8FD")
        .style("border-bottom", "1px solid white")
        .style("border-right", "1px solid white")
        .style("border-left", "1px solid white")


      // body text
      // let text = node.summary
        // node.type === "start"
        //   ? "Start"
        //   : node.type === "end"
        //   ? "End"
        //   : !node.approvers || node.approvers.length === 0
        //   ? "No approver"
        //   : node.approvers.length > 1
        //   ? `${node.approvers[0].name + "..."}`
        //   : node.approvers[0].name;
      // let bodyTextY;
      // if (node.type !== "start" && node.type !== "end") {
        // if (node.id === 3) {
        //   bodyTextY = node.y + 25;
        // } else {
          // bodyTextY = node.y + 65 + roundTo20(node.height - 20) / 2;
        // }
      // } else {
      //   bodyTextY = node.y + 5 + roundTo20(node.height) / 2;
      // }

      g.append('foreignObject')
        .attr("x", node.x)
        .attr("y", node.y + node.height / 2)
        .attr("class", "unselectable")
        .style("width", node.width + "px")
        .style("height", node.height / 2 + "px")
        .append('xhtml:p')
        // .append('p')
        .style("width", node.width + "px")
        .style("height", node.height / 2 + "px")
        .style("box-sizing", "border-box")
        .style("padding", "4px 8px 4px 8px")
        .style("color", "white")
        .style("margin", 0)
        // .style("word-wrap", "break-word")
        .style("overflow-wrap", "break-word")
        // .style("color", "red")
        // .html('test test test   test test testtesttesttesttesttesttesttest')
        .text(() => node.summary)

      // g.append("text")
      //   .attr("x", node.x + 4)
      //   // .attr("y", bodyTextY)
      //   .attr("y", node.y + node.height / 2 + 20)
      //   .attr("class", "unselectable")
      //   // .attr("text-anchor", "middle")
      //   .text(function () {
      //     return text;
      //   })
      //   .each(function wrap() {
      //     let self = d3.select(this),
      //       textLength = self.node().getComputedTextLength(),
      //       text = self.text();
      //     while (textLength > node.width - 2 * 4 && text.length > 0) {
      //       text = text.slice(0, -1);
      //       self.text(text + "...");
      //       textLength = self.node().getComputedTextLength();
      //     }
      //   });
    },
  },
};
</script>
<style scoped>
#toolbar {
  margin-bottom: 10px;
}

.title {
  margin-top: 10px;
  margin-bottom: 0;
}

.subtitle {
  margin-bottom: 10px;
}

#toolbar > button {
  margin-right: 4px;
}

.container {
  /* width: 800px; */
  width: 96%;
  margin: auto;
  height: 80vh;
}
</style>
