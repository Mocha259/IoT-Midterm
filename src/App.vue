<template>
  <div id="app">
    <!-- 画布背景 -->
    <div class="background">
      <canvas id="lines"></canvas>
    </div>
    <!-- div 前景 -->
    <div class="front" >
      <!-- 传感器网络区域 -->
      <dv-border-box-8 class="node-area">
        <el-checkbox-group v-model="checkList" @change="changeNet">
          <div v-for="r in row" :key="r" class="checkboxNode"> 
            <el-checkbox v-for="c in 12" :key="c" :label="r + c" :ref="r + c" :id=" r + c" ></el-checkbox>
           </div>
        </el-checkbox-group>
      </dv-border-box-8>

      <!-- 连接输入区域和网络区域的装饰 -->
      <dv-decoration-2 class="decoration-2" />
      
      <!-- 距离优化路径 -->
      <dv-border-box-8 class="path-optimal"><h3 style="position: absolute; left: 10px; top: 5px">最短路径：{{bestDist}}m</h3><br><h4 style="position: absolute; left: 10px; top: 30px">路径：</h4>
        <div style="margin-left: 10px; margin-top: 30px; ">
          <!-- 1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111 -->
          <!-- <span style="margin-left: 10px; margin-top: 30px; overflow-x: scroll; overflow-y: hidden" v-for="item in optimPath" :key="item">{{item}} -></span> -->
          {{optimPath}}
        </div>
      </dv-border-box-8>

      <!-- 连接优化路径和输入区域的装饰 -->
      <!-- <dv-decoration-2 :reverse="true" class="decoration-path"  style="position: absolute; top: 20%; right: 0px; height: 200px;" /> -->

      <dv-border-box-8 class="input-area">
        <div class="leftSpeed" style="float: left; width: 40%; margin-top: 20px;">
          <span style="position: absolute; left: 17px; top: 20px font-size: 20px; color: white">速度：</span>
        </div>
        <div class="speed">
          <el-input 
          placeholder="请输入速度"
          v-model="speed"
          clearable>
          </el-input>
        </div><br>
        <div class="leftSpeed" style="float: left; width: 40%; margin-top: 45px;">
          <span style="position: absolute; left: 17px; top: 15px font-size: 20px; color: white">间隔：</span>
        </div>
        <div class="interval"><el-input
          placeholder="请输入间距"
          v-model="interval"
          clearable>
        </el-input></div>
        <div class="leftInterval" style="float: left; width: 40%; margin-top: 95px;">
          <span style="position: absolute; left: 7px; top: 117px; font-size: 14px; color: white">起点坐标：</span>
        </div>
        <div class="startRow">
          <el-select v-model="rowOfStart" placeholder="纵坐标" @change="changeNet">
            <el-option
              v-for="item in selectedRow"
              :key="item"
              :label="item"
              :value="item">
            </el-option>
          </el-select>
        </div>
        <div class="startCol">
          <el-select v-model="colOfStart" placeholder="横坐标"  @change="changeNet">
            <el-option
              v-for="item in selectedCol"
              :key="item"
              :label="item"
              :value="item">
            </el-option>
          </el-select>
        </div>

        <el-button class="start" @click="start">
          开始优化
        </el-button>
        <el-button class="reset" @click="reset">
          重置
        </el-button>
      </dv-border-box-8>

      <!-- 连接优化路径和输入区域的装饰 -->
      <!-- <dv-decoration-2 :reverse="true" class="decoration-time" /> -->
      
      <!-- 时间优化路径 -->
      <dv-border-box-8 class="time-optimal"><h3 style="position: absolute; left: 10px; top: 5px">最短时间：{{bestTime}}s</h3>
      <br><h4 style="position: absolute; left: 10px; top: 30px">路径：</h4>
        <div style="margin-left: 10px; margin-top: 30px; ">
          {{optimTime}}
        </div>
      </dv-border-box-8>

    </div>
    <div class="drawPath">
      <canvas id="Path"></canvas>
    </div>
  </div>
</template>

<script>
export default {

   data() {
    return{
        
        speed: '',
        // realSpeed: parseFloat(speed),
        interval: '5',
        // realIntercal: parseInt(interval),
         isRouterAlive: true,
        input: '',
        checkList: [],//记录选中的结点
        graphInfo: [],
        row: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'],
        selectedRow: [],
        selectedCol: [],
        rowOfStart: '',
        colOfStart: '',
        rowOfEnd: '',
        colOfEnd: '',
        start_node: '',
        end_node: '',
        optimPath: ['A1','A2','A3','A4','A5','A6','B1','B2','B3'],
        optimTime: ['A1','B2','C3','D4','E5','F5','G5','H5','H6','G6','F6','E6','D7','C8','B9','A10','A8','A7','A6','A5'],
        bestDist: 99999,
        bestTime: 99999,
        arrow: '-->',
        tmp: {},
      }
    },
    methods: {
      permutation(a) {
          // 保存最终输出结果
        let result = [];
        
        // 定义 m 值默认等于 n，即全排列
        let m = a.length;
        
        // 定义递归函数保存结果到数组中
        // _a 为输入数组，
        // tmpResult 为保存单个情况结果的数组
        function recur(_a, tmpResult = []) {
            if (tmpResult.length === m) {
            
                // 结果达到 m 个时保存结果，
                // 停止递归并进入下一次遍历
                result.push(tmpResult);
                
            } else {
                for (let i = 0; i < _a.length; i++) {
                    
                    // 复制一份输入数组，防止引用值被改变
                    let tmpA = _a.concat();
                
                    // 复制一份保存结果的数组，防止每次遍历相互影响
                    let _tmpResult = tmpResult.concat();
                    
                    // 保存当前遍历值
                    _tmpResult.push(tmpA[i]);
                    
                    // 删除当前遍历值，传递参数进入下一层递归
                    tmpA.splice(i, 1);
                    recur(tmpA, _tmpResult);
                }
            }
        }
        // 开始执行递归，然后返回最后结果
        recur(a);
        return result;
      },
      function(){
        let self = this
        class Graph {
          constructor() {
              this.edges = {};
              this.nodes = [];
              this.all_path = [];
          }

          addNode(node) {
              this.nodes.push(node);
              this.edges[node] = [];
          }

          addEdge(node1, node2, weight = 1) {
              this.edges[node1].push({ node: node2, weight: weight });
              this.edges[node2].push({ node: node1, weight: weight });
          }

          addDirectedEdge(node1, node2, weight = 1) {
              this.edges[node1].push({ node: node2, weight: weight });
          }

          display() {
              let graph = "";
              this.nodes.forEach(node => {
                graph += node + "->" + this.edges[node].map(n => n.weight + '-' + n.node ).join(", ") + "\n";
              });
              console.log(graph);
          }

          getDistOptimalPath() {
            //暂时默认第一个选中的点是起点，最后一个选中的点是终点
            var start_node  = self.checkList[0]
            var end_node    = self.checkList[self.checkList.length - 1]

            for(var i = 0; i < this.nodes.length; ++i){
              if(this.nodes[i] == start_node 
                || this.nodes[i] == end_node)
              {
                this.nodes.splice(i, 1)
              }
            }
            
          }


        }
        // 先保存起点和终点
        this.start_node = this.checkList[0]
        this.end_node   = this.checkList[this.checkList.length - 1]
        // 创建图并添加点和边
        var graph = new Graph()
        for(var i = 0; i < this.checkList.length; i++){
            graph.addNode(this.checkList[i])
        }
        for(var i = 0; i < this.graphInfo.length; i++){
          graph.addEdge(this.graphInfo[i].start, this.graphInfo[i].end, this.graphInfo[i].dis)
        }
        //剔除起点和终点，对中间结点全排列
        graph.getDistOptimalPath()
        this.all_path = this.permutation(graph.nodes)
        // 把起点和终点放回去
        var best_dist = 999999
        var best_path_index = 0
        for(var i = 0; i < this.all_path.length; i++){
          this.all_path[i].push(this.end_node)
          this.all_path[i].unshift(this.start_node)
          
          var tmp_dist = 0
          for(var j = 0; j < this.all_path[i].length - 1; j++){
            if(tmp_dist > best_dist){
              continue
            }
            for(var k = 0; k < graph.edges[this.all_path[i][j]].length; k++){
              if(graph.edges[this.all_path[i][j]][k].node == this.all_path[i][j + 1]){
                tmp_dist += graph.edges[this.all_path[i][j]][k].weight
                break
              } //endif
            }
            
          }//end of for loop
          console.log('tmp_dist')
          console.log(best_dist)
          if (tmp_dist < best_dist){
            best_dist = tmp_dist
            best_path_index = i
          }//endif
        }

        this.bestDist = best_dist
        this.optimPath.splice(0, this.optimPath.length)
        for(var i = 0; i < this.all_path[best_path_index].length; i++){
          this.optimPath.push(this.all_path[best_path_index][i])
        }
        console.log('best path: ' + this.optimPath)
        console.log('best dist: ' + best_dist)
        // graph.display()
        // console.log('所有结点：' + )
        // console.log('所有中间结点：' + graph.nodes)
        
      },
      getGraphInfo() {
        for(var i = 0; i < this.checkList.length - 1; i++){
          for( var j = i + 1; j < this.checkList.length; j++){
            var linkOf2Node = { start: '', dis: -1.0, end: '' }
            linkOf2Node.start = this.checkList[i]
            linkOf2Node.end   = this.checkList[j]
            
            var startX = linkOf2Node.start.slice(1)
            var endX   = linkOf2Node.end.slice(1)
            var disOnX = Math.abs(parseInt(endX) - parseInt(startX))
            
            var startY  = linkOf2Node.start.slice(0,1)
            var endY    = linkOf2Node.end.slice(0,1)
            var disOnY = Math.abs(startY.charCodeAt() - endY.charCodeAt())

            var ans = 0
            if(disOnX < disOnY){
              ans = Math.sqrt(2 * disOnX * disOnX) + disOnY - disOnX
            }else{
              ans = Math.sqrt(2 * disOnY * disOnY) + disOnX - disOnY
            }
            linkOf2Node.dis = ans * this.interval
            this.graphInfo.push(linkOf2Node)
          }
        }
      },
      getElementPagePosition(element){
        //计算x坐标
        var actualLeft = element.offsetLeft;
        var current = element.offsetParent;
        while (current !== null){
          actualLeft += current.offsetLeft;
          current = current.offsetParent;
        }
        //计算y坐标
        var actualTop = element.offsetTop;
        var current = element.offsetParent;
        while (current !== null){
          actualTop += (current.offsetTop+current.clientTop);
          current = current.offsetParent;
        }
        //返回结果
        return {x: actualLeft, y: actualTop}
      },
      getElementViewPosition(element){
        //计算x坐标
        var actualLeft = element.offsetLeft;
        var current = element.offsetParent;
        while (current !== null){
          actualLeft +=  (current.offsetLeft+current.clientLeft);
          current = current.offsetParent;
        }
        if (document.compatMode == "BackCompat"){
          var elementScrollLeft=document.body.scrollLeft;
        } else {
          var elementScrollLeft=document.documentElement.scrollLeft;
        }
        var left = actualLeft - elementScrollLeft;
        //计算y坐标
        var actualTop = element.offsetTop;
        var current = element.offsetParent;
        while (current !== null){
          actualTop += (current.offsetTop+current.clientTop);
          current = current.offsetParent;
        }
        if (document.compatMode == "BackCompat"){
          var elementScrollTop=document.body.scrollTop;
        } else {
          var elementScrollTop=document.documentElement.scrollTop;
        }
        var right = actualTop-elementScrollTop;
        //返回结果
        return {x: left, y: right}
      },
      changeNet(){
        //清空
        this.selectedRow.splice(0, this.selectedRow.length)
        this.selectedCol.splice(0, this.selectedCol.length)
        for(var i = 0; i < this.checkList.length; i++){
          var tmpRow = this.checkList[i].charAt(0)
          if(this.selectedRow.indexOf(tmpRow) < 0){
            this.selectedRow.push(tmpRow)
          }
          var tmpCol = this.checkList[i].slice(1)
          var col = parseInt(tmpCol)
          if(this.selectedCol.indexOf(col) < 0){
            this.selectedCol.push(col)
          }
        }
        this.selectedRow.sort();
        this.selectedCol.sort();
      },
      reset(){
          this.checkList.slice(0, this.checkList.length)
          this.selectedRow.splice(0, this.selectedRow.length)
          this.selectedCol.splice(0, this.selectedCol.length)
          this.speed = ''
          this.interval = ''
          location.reload();
      },
      start(){
          this.getGraphInfo()
          // console.log(this.graphInfo)
          this.function()
          console.log(this.tmp)
          var canvas = document.getElementById("Path")
          var width = window.innerWidth
          var height = window.innerHeight
          canvas.width = width
          canvas.height = height
                       
          //检测当前浏览器是否支持Canvas对象，以免在一些不支持html5的浏览器中提示语法错误
          if(canvas.getContext){
            var offset_x = 3.5
            var offset_y = 6.5
            //97 164
            //365 500
            for(var i = 0; i < this.optimPath.length - 1; i++){
              var element = document.getElementById(this.optimPath[i])
              var start_x = this.getElementPagePosition(element).x + offset_x
              var start_y = this.getElementPagePosition(element).y + offset_y
              // var start_x = this.getElementViewPosition(element).x + offset_x
              // var start_y = this.getElementViewPosition(element).y + offset_y
              element = document.getElementById(this.optimPath[i + 1])
              var end_x = this.getElementPagePosition(element).x + offset_x
              var end_y = this.getElementPagePosition(element).y + offset_y
              // var end_x = this.getElementViewPosition(element).x + offset_x
              // var end_y = this.getElementViewPosition(element).y + offset_y
              
              // console.log('起点：(' + start_x + ',' + start_y + ')-->终点：('+ end_x + ',' + end_y + ')')
              var ctx = canvas.getContext("2d")
              ctx.beginPath()
              
              ctx.strokeStyle = `rgba(${81}, ${160}, ${255},${0.9})`
              ctx.lineCap = "round"
              ctx.lineJoin = "round"
              ctx.lineWidth = 5
              ctx.shadowColor = 'rgba(255,255,0,1)';
              ctx.shadowBlur = 20;
              ctx.shadowOffsetX = 10100;
              ctx.shadowOffsetY = 10100;
              ctx.moveTo(start_x, start_y)
              ctx.lineTo(end_x, end_y)
              ctx.stroke()
              ctx.closePath()

              // ctx.strokeStyle = red
              // ctx.lineWidth = 2
              // ctx.moveTo(97, 164)
              // ctx.lineTo(365, 500)
              // ctx.stroke()
              // let next_x
              // let next_y

              // // 第一帧执行的时间
              // let startTime;
              // // 期望动画持续的时间
              // const duration = 1000
              // var ctx = canvas.getContext("2d")
              // ctx.beginPath()
              // ctx.moveTo(start_x, start_y)
              // ctx.strokeStyle = `rgba(${81}, ${160}, ${255},${0.25})`
              // ctx.lineWidth = 4
              // const step = (currentTime) => {
              //   // 第一帧绘制时记录下开始的时间
              //   !startTime && (startTime = currentTime)
              //   // 已经过去的时间(ms)
              //   const timeElapsed = currentTime - startTime
              //   // 动画执行的进度 {0,1}
              //   const progress = Math.min(timeElapsed / duration, 1)

              //   // 绘制方法
              //   const draw = () => {
              //       // 计算这一帧中线段应该到达的坐标点
              //       next_x = start_x + (end_x - start_x) * progress
              //       next_y = start_y + (end_y - start_y) * progress
              //       // 用直线连接子路径的最后的点到(nextX,nextY)坐标
              //       ctx.lineTo(next_x, next_y)
              //       // 绘制路径(所有子路径都会被绘制一次)
              //       ctx.stroke()
              //   }
              //   draw()

              //   if (progress < 1) {
              //       requestAnimationFrame(step)
              //   } else {
              //       console.log('动画执行完毕')
              //   }
              // }

              // requestAnimationFrame(step)
              ctx.closePath()
              //沿着坐标点顺序的路径绘制直线
            } 
            
          }   
      },

    },
    mounted() {
      console.log('Page has been refreshed.');
      (function () {
        const canvas = document.getElementById("lines");
        const ctx = canvas.getContext("2d");
        let width;
        let height;
        class Line {
          constructor(origin, size, length, color, style = "pattern") {
          this.size = size;
          this.origin = origin;
          this.length = length;
          this.color = color;
          this.style = style;
          this.origin = `M${origin.x},${origin.y}`;
          this.offSet = 0;
          this.line = null;
          this.offSetSpeed = length / size;
          }
          getColorString() {
            return `hsla(${this.color.h}deg,${this.color.s}%,${this.color.l}%,${this.color.a})`;
          }
          generators() {
            return [
              {
                line: `h${this.size}`,
                mag: this.size
              },
              {
                line: `h-${this.size}`,
                mag: this.size
              },
              {
                line: `v${this.size}`,
                mag: this.size
              },
              {
                line: `v-${this.size}`,
                mag: this.size
              },
              {
                line: `l${this.size},${this.size}`,
                mag: Math.hypot(this.size, this.size)
              },
              {
                line: `l${this.size}-${this.size}`,
                mag: Math.hypot(this.size, this.size)
              },
              {
                line: `l-${this.size},${this.size}`,
                mag: Math.hypot(this.size, this.size)
              },
              {
                line: `l-${this.size}-${this.size}`,
                mag: Math.hypot(this.size, this.size)
              }
            ];
          }
        generate() {
          let segments = this.generators(this.size);
          let path = this.origin;
          let mag = 0;
          let fragment;
          let i;
          for (i = 0; i < this.length; i += 1) {
            fragment = segments[(Math.random() * segments.length) | 0];
            path += ` ${fragment.line}`;
            mag += fragment.mag;
          }
          this.line = {
            path,
            mag
          };
          return this;
        }
        renderStyle(style) {
          if (style === "glitches") {
            ctx.lineDashOffset = this.line.mag + this.offSet;
            ctx.setLineDash([
              this.size ** 1.5,
              (this.line.mag / this.length) * this.size ** 2
            ]);
            this.offSet += 20;
            // this.size / (this.size ** 2);
            ctx.lineWidth = 2;
            return this;
          }
          if (style === "pattern") {
            ctx.lineDashOffset = this.line.mag - this.offSet;
            ctx.setLineDash([this.line.mag, this.line.mag]);
            this.offSet += 10;
            //this.size / (this.size ** 100);
            ctx.lineWidth = 0.2;
          }
        }
        mutatePath() {
          let lineFragment = this.line.path.split(" ").slice(1);
          let generator = this.generators();
          lineFragment[(Math.random() * lineFragment.length) | 0] =
            generator[(Math.random() * generator.length) | 0].line;
          this.line.path = `${this.line.path.split(" ")[0]} ${lineFragment.join(
            " "
          )}`;
        }
        draw() {
          !this.line && this.generate();

          ctx.strokeStyle = this.getColorString();
          this.renderStyle(this.style);
          ctx.lineCap = "round";
          ctx.lineJoin = "round";
          ctx.stroke(new Path2D(this.line.path));
          return this;
        }
      }
      function clear() {
        ctx.fillStyle = `hsla(200deg, 20%, 10%, 0.3)`;
        ctx.fillRect(0, 0, width, height);
      }
      function generateLines(amount) {
        let lines = [];
        let styles = [
          {
            size: 1.25,
            style: "pattern",
            color: { h: 210, s: 100, l: 70, a: 0.5 }
          },
          { size: 2.5, style: "pattern", color: { h: 190, s: 90, l: 50, a: 0.3 } },
          { size: 5, style: "pattern", color: { h: 210, s: 70, l: 60, a: 0.2 } },
          { size: 10, style: "pattern", color: { h: 310, s: 80, l: 55, a: 0.15 } },
          { size: 20, style: "pattern", color: { h: 200, s: 25, l: 35, a: 0.12 } },
          { size: 20, style: "pattern", color: { h: 210, s: 20, l: 40, a: 0.12 } },
          { size: 40, style: "pattern", color: { h: 190, s: 40, l: 50, a: 0.12 } },
          { size: 80, style: "pattern", color: { h: 220, s: 50, l: 60, a: 0.12 } },
          { size: 40, style: "glitches", color: { h: 300, s: 100, l: 50, a: 0.3 } },
          { size: 20, style: "glitches", color: { h: 210, s: 100, l: 50, a: 0.3 } },
          { size: 60, style: "glitches", color: { h: 30, s: 100, l: 50, a: 0.3 } }
        ];
        for (let i = 0; i < amount; i += 1) {
          let style = styles[(Math.random() ** 2 * styles.length) | 0];
          lines.push(
            new Line(
              { x: width * 0.5, y: height * 0.5 },
              style.size,
              500 + Math.random() * 1000,
              style.color,
              style.style
            )
          );
        }
        return lines;
      }
      let id;
      function resize() {
        id = cancelAnimationFrame(id);
        width = window.innerWidth;
        height = window.innerHeight;
        canvas.width = width;
        canvas.height = height;
        const lines = generateLines(40);
        function update() {
          if (!(id % 3)) {
            clear();
            lines.forEach((line) => {
              line.draw();
              if (!(id % 5) && Math.random() > 0.95) {
                line.mutatePath();
              }
            });
          }
          id = requestAnimationFrame(update);
        }
        id = requestAnimationFrame(update);
      }
      window.addEventListener("resize", resize, {
        passive: true
      });
      resize();})();
    }
}
</script>


<style scoped>

.app {
  min-width:1280px;
}

.background {
    width:100%;  
    height:100%;
    z-index:-999;
    position: absolute;
    top: 0px;
    left: 0px;
    bottom: 0px;
    right: 0px;
    display: flex;

}

.front {
  height: 100%;
  width: 100%;
  z-index: 999;

}

.input-area{
  position: absolute;
  top:36%;
  right: 5%;
  height: 30%;
  width: 10.5%;
  /* width: 200px; */
  padding: 0px;
  border: 3px solid 	#87CEFA;
  
  /* left: 1450px; */
}

.speed {
  position: absolute;
  top: 3%;
  left: 33%;
  width: 60%;
  height: 20%;
  /* margin-top: 10px; */
  border-radius: 15%;
}

.interval {
  position: absolute;
  left: 33%;
  width: 60%;
  margin-top: 18%;
  border-radius: 15%;
}

.startRow {
  position: absolute;
  left: 37%;
  /* width: 110px; */
  width: 56%;
  /* margin-top: 85px; */
  margin-top: 43%;
  border-radius: 15%;
}

.startCol {
  position: absolute;
  left: 37%;
  width: 56%;
  margin-top: 68%;
  /* margin-top: 131px; */
  border-radius: 15%;
}

.node-area {
  position: absolute;
  top: 10%;
  left: 2%;
  width: 60%;
  height: 80%;
  border: 3px solid 	#87CEFA;
  z-index: 2;
}

.el-input>>> .el-input__inner {
    background-color: transparent;
    box-shadow: 0 0 5px #87CEFA;
    color:#87CEFA
}

.el-select>>> .el-input__inner  {
  background-color: transparent !important;
  box-shadow: 0 0 5px #87CEFA;
  color:#87CEFA
}


.el-checkbox-group {
  position: absolute;
  top: 9%;
  left: 5%;
}


.el-checkbox-group{
  height: 100%;
  width: 100%;
}

.checkboxNode {
  height: 11.5%;
  width: 100%;
}

.el-checkbox{
  width: 5%;
  height: 5%;
}

.decoration-2 {
  position: absolute;
  right: 15.5%;
  top: 50%;
  width:22.5%;
  height:10px;
  box-shadow: 0 0 5px #87CEFA;
  transform: rotate(180deg);
}

.path-optimal {
  position: absolute;
  top: 10%;
  right: 1%;
  height: 10%;
  width: 35%;
  border: 3px solid 	#87CEFA;
  color: white;
}

.time-optimal {
  position: absolute;
  bottom: 10%;
  right: 1%;
  height: 11.5%;
  width: 35%;
  border: 3px solid 	#87CEFA;
  color: white;
  font-size: 100%;
}


* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font-family: "Roboto";
  font-weight: 100;
}

body {
  font-size: 18px;
  color: hsla(210deg, 100%, 100%, 1);
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

h1 {
  text-transform: uppercase;
  letter-spacing: 1.5em;
  font-size: clamp(1em, 5vw, 4em);
  animation: breath 10000ms ease-in-out infinite alternate;
}
h1 > .last {
  letter-spacing: 0;
}

@keyframes breath {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(1.5);
  }
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  margin: 0;
  padding: 0;
  background-color: hsla(240deg, 20%, 20%, 1);
  height: 100%;
  width: 100%;
  /* animation: breath 20s infinite alternate */
}

.start {
  position: absolute; 
  /* bottom: 10px; 
  left: 15px;  */
  bottom: 10%;
  left: 8%;
  background: transparent; 
  box-shadow: 0 0 5px #87CEFA; 
  font-size: 80%; 
  /* height: 30px !important; */
  /* width: 80px; */
  height: 12%;
  width: 50%;
  color: white
}

.reset {
  position: absolute; 
   bottom: 10%;
  left: 57%;
  background: transparent; 
  box-shadow: 0 0 5px #87CEFA; 
  font-size: 80%; 
  color: white;
  height: 12%;
  width: 30%;
}

.el-button:hover {
  color: #7bc5f0 !important;
  background: transparent;
}

.start, .start:focus:not(.start:hover){ 
    background: transparent;
    color: white;
}

.reset, .reset:focus:not(.reset:hover){ 
    background: transparent;
    color: white;
}

.el-checkbox {
  /* margin-top: 20px; */
  margin-left: 25px;;
}

.drawPath {
  position: absolute;
  /* top: 0;
  left: 0;
  margin: 0; */
  /* padding: 0; */
  z-index: -1;
  width: 100%;
  height: 100%;
}

.drawPath canvas {
  width: 100%;
  height: 100%;
  top: 0px;
  left: 0px;
  position: absolute;
  /* top: 0;
  left: 0;
  margin: 0;
  padding: 0; */
  /* z-index: -1; */
  background-color:transparent;
  display: flex;

  /* background: #21e251; */
}

</style>

<style>
.el-select-dropdown{
    /* background-color: transparent !important; */
    border-radius: 10px;
     /* filter: blur(20px);值越大越模糊 */
  }

  .el-select-dropdown__item{
    /* background-color: transparent !important; */
    /* background-color: rgba(255, 255, 255, 0.9); */
    color:#7bc5f0 !important;
    font-weight: bolder;
    /* filter: blur(20px); */
  }

.el-select-dropdown__item:hover{
    color:#409eff !important;
    background-color: transparent !important;
    box-shadow: 0 0 10px rgb(73, 201, 240);
  } 

.el-checkbox__input.is-checked .el-checkbox__inner, .el-checkbox__input.is-indeterminate .el-checkbox__inner {
  background-color: rgb(4, 240, 44) !important;
  border-radius: 50%;
  box-shadow: 0 0 10px rgb(4, 240, 44);
  border-color: #21e251;
  content: "" !important;
  background-image: none !important;
  animation:warn 1s infinite alternate;
  transform: rotate(360deg) !important;
}

.el-checkbox__inner::after{
  -webkit-box-sizing:content-box;
  box-sizing:content-box;content:"";
  border:0px solid #21e251 !important;
}

@keyframes warn {
        0% {
            transform: scale(1);
            opacity: 0.9;
        }
        20% {
            transform: scale(1);
            opacity: 0.8;
        }
        40% {
            transform: scale(1.2);
            opacity: 0.8;
        }
        60% {
            transform: scale(1.4);
            opacity: 0.8;
        }
        80% {
            transform: scale(1.6);
            opacity: 0.8;
        }
        100% {
            transform: scale(1.8);
            opacity: 0.8;
        }
    }

@keyframes warn_2 {
        0% {
            transform: scale(1);
            opacity: 0.9;
        }
        20% {
            transform: scale(1);
            opacity: 0.8;
        }
        40% {
            transform: scale(1.2);
            opacity: 0.8;
        }
        60% {
            transform: scale(1.2);
            opacity: 0.8;
        }
        80% {
            transform: scale(1.2);
            opacity: 0.8;
        }
        100% {
            transform: scale(1.2);
            opacity: 0.8;
        }
    }
.el-checkbox__input .el-checkbox__inner{
  border-color:  #a70b18e0;
  background-color:#a70b18e0;
  box-shadow: 0 0 10px rgb(207, 19, 19);
  border-radius: 50%;
  animation:warn_2 1s infinite alternate;
}


.el-checkbox__input.is-checked + .el-checkbox__label {
  background-color: transparent;
  background:transparent;
  color: #21e251 !important;
  content: none !important;

}
.el-checkbox.is-bordered.is-checked{
  background-color:#21e251;
  box-shadow: 0 0 20px #21e251;
  border-color: #21e251;
  
}
.el-checkbox__input.is-focus .el-checkbox__inner{
  border: transparent !important;
}

</style>