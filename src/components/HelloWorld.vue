<template>
  <div class="hello">
    <div v-if="entering_param">
      <div class="game-over">请输入贪吃蛇的速度(1~10)</div>
      <div class="input-btn">
        <input type="number" v-model="enter_speed" />
        <button @click="sure">确定</button>
      </div>
    </div>
    <div v-if="!entering_param">
      <div class="game-over" v-if="game_over">
        <div>游戏结束!</div>
        <div>最终得分：{{score}}</div>
        <div><button @click="playAgain">再来一局</button></div>
      </div>
      <div class="score" v-if="!game_over">
        <div>
          <p>WASD或方向键控制方向</p>
          <p>按空格暂停/开始</p>
          <p>按E加速，按Q减速（消耗1长度）</p>
        </div>
        <span>得分：{{score}}</span>
      </div>
      <div v-if="!game_over">
        <div class="line" v-for="(v1, i1) in chess_map" :key="'o' + i1">
          <span
            :class="{'box': 1, 'active-box': v2 === 1, 's-head': v2 === 2, 'apple': v2 === 3}"
            v-for="(v2, i2) in v1"
            :key="'i' + i2"
          ></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      cur_direction: "right",
      new_direction: "right",
      chess_map: [], // 地图数组
      snake_pos_arr: [], // 蛇的身体每一格的位置坐标
      game_over: false,
      counter: "",
      speed: 50,
      apple_pos: null, // 苹果的位置
      score: 0,
      entering_param: true,
      enter_speed: 5,
      is_pausing: false,
    };
  },
  mounted() {
    document.onkeydown = this.keydown
    this.init();
  },
  methods: {
    // 初始化
    init() {
      this.game_over = false
      this.cur_direction = "right"
      this.new_direction = "right"
      this.score = 0
      this.chess_map = new Array(50);
      this.chess_map.fill([]);
      this.chess_map = this.chess_map.map(() => {
        let arr = new Array(50);
        arr.fill(0);
        return arr;
      });
      this.chess_map[0][0] = 1;
      this.chess_map[0][1] = 1;
      this.chess_map[0][2] = 1;
      this.chess_map[0][3] = 1;
      this.chess_map[0][4] = 2;
      this.snake_pos_arr = [
        [0, 0],
        [0, 1],
        [0, 2],
        [0, 3],
        [0, 4],
      ];
    },
    playAgain(){
      this.init()
      this.sure()
    },
    sure(){
      console.log(this.enter_speed);
      if( this.enter_speed > 10 || this.enter_speed < 1){
        window.alert("请输入1到10的数字")
        return
      }
      this.speed = 550 - this.enter_speed*50
      this.entering_param = false
      this.counter = setTimeout(() => {
        this.go();        
      }, this.speed);
    },
    go() {
      // 生成苹果
      this.genApple()

      // 计算头部位置
      let newhead = this.calSnakeHeadPosition()

      // 重置当前位置，判断是否吃到了苹果
      this.resetSnakePosition(newhead)

      // 判断游戏是否结束
      if (this.checkGameEnd(newhead)) {
        this.game_over = true
        this.apple_pos = null
        clearTimeout(this.counter)
        return;
      }
  
      // 清空地图并重新填
      this.reDrawMap()

      // 执行下一个轮回
      this.counter = setTimeout(() => {
        this.go();
      }, this.speed);
    },
    // 计算蛇的头部位置
    calSnakeHeadPosition(){
      let newhead = [...this.snake_pos_arr[this.snake_pos_arr.length - 1]];
      switch (this.new_direction) {
        case "right":
          newhead[1] = newhead[1] + 1;
          break;
        case "left":
          newhead[1] = newhead[1] - 1;
          break;
        case "up":
          newhead[0] = newhead[0] - 1;
          break;
        case "down":
          newhead[0] = newhead[0] + 1;
          break;
        default:
          break;
      }
      return newhead
    },
    // 重新绘制地图数组
    reDrawMap(){
      this.chess_map.fill([]);
      this.chess_map = this.chess_map.map(() => {
        let arr = new Array(50);
        arr.fill(0);
        return arr;
      });
      this.snake_pos_arr.forEach((v, i) => {
        if(i === this.snake_pos_arr.length - 1){
          this.chess_map[v[0]][v[1]] = 2;
        } else{
          this.chess_map[v[0]][v[1]] = 1;
        }
      });
      if(this.apple_pos){
        this.chess_map[this.apple_pos[0]][this.apple_pos[1]] = 3
      }
      // 改变当前状态的方向
      this.cur_direction = this.new_direction;
    },
    // 重置蛇的位置数组，判断是否吃到苹果
    resetSnakePosition(newhead){
      this.snake_pos_arr.push(newhead);
      // 如果头的位置和苹果位置重合，就吃到了
      if(this.apple_pos[0] !== newhead[0] || this.apple_pos[1] !== newhead[1]){
        this.snake_pos_arr.splice(0, 1);
      }else{
        this.apple_pos = null
        this.score++
      }
    },
    // 检查游戏是否结束
    checkGameEnd(newhead) {
      let res = false;
      // 判断是否碰墙
      if (newhead[0] >= 50 || newhead[1] >= 50 || newhead[0] < 0 || newhead[1] < 0) {
        res = true;
      }
      // 判断是否碰自己
      for(let i = 0; i < this.snake_pos_arr.length - 2; i++){
        if(this.snake_pos_arr[i][0] === newhead[0] && this.snake_pos_arr[i][1] === newhead[1]){
          res = true;
          break
        }
      }
      return res;
    },
    // 生成苹果
    genApple(){
      // 如果还有苹果，就不生成新苹果
      if(this.apple_pos){
        return
      }
      // 计算空余的位置
      let empty_pos = []
      this.chess_map.forEach((v1,k1)=>{
        v1.forEach((v2,k2)=>{
          if(v2 !==1 && v2 !== 2 && v2 !== 3){
            empty_pos.push([k1, k2])
          }
        })
      })
      let index = Math.ceil(Math.random()*empty_pos.length)
      this.apple_pos = empty_pos[index]    
    },
    keydown(e) {
      // 按空格暂停
      console.log(e);
      if(e.key === " "){
        if(this.is_pausing){
          this.is_pausing = false
          this.go()
        }else{
          this.is_pausing = true
          clearTimeout(this.counter)
        }
        return
      }
      if (this.cur_direction === "left" && (e.key === "d" || e.key === "ArrowRight")) {
        return;
      }
      if (this.cur_direction === "right" && (e.key === "a" || e.key === "ArrowLeft")) {
        return;
      }
      if (this.cur_direction === "up" && (e.key === "s" || e.key === "ArrowDown")) {
        return;
      }
      if (this.cur_direction === "down" && (e.key === "w" || e.key === "ArrowUp")) {
        return;
      }
      if (e.key === "w" || e.key === "ArrowUp") {
        this.new_direction = "up";
      }
      if (e.key === "s" || e.key === "ArrowDown") {
        this.new_direction = "down";
      }
      if (e.key === "a" || e.key === "ArrowLeft") {
        this.new_direction = "left";
      }
      if (e.key === "d" || e.key === "ArrowRight") {
        this.new_direction = "right";
      }
      // 加速功能
      if (e.key === "e") {
        // 只有长度大于2的时候可以加速，加速会消耗一个长度
        if(this.snake_pos_arr.length > 2){
          this.speed = 30
          clearTimeout(this.counter)
          this.snake_pos_arr.splice(0, 1)
          this.go()
          setTimeout(()=>{
            clearTimeout(this.counter)
            this.sure()
          }, 500)
        }
      }
      if (e.key === "q") {
        // 只有长度大于2的时候可以加速，加速会消耗一个长度
        if(this.snake_pos_arr.length > 2){
          this.speed = 500
          clearTimeout(this.counter)
          this.snake_pos_arr.splice(0, 1)
          this.go()
          setTimeout(()=>{
            clearTimeout(this.counter)
            this.sure()
          }, 1500)
        }
      }
    },
  },
};
</script>

<style scoped lang="less">
.hello {
  width: 500px;
  height: 500px;
  background-color: #777;
  margin: 0 auto;
}
.line {
  display: flex;
  box-sizing: border-box;
  height: 10px;
  // border-top: 1px solid #777;
}
.box {
  box-sizing: border-box;
  width: 10px;
  height: 10px;
  background-color: #333;
  transition: all 0.1s;
  // border-right: 1px solid #777;
}
.active-box {
  background-color: #f00 !important;
}
.s-head{
  background-color: #f0f !important;
}
.apple{
  background-color: #fff !important;
}
.game-over {
  font-size: 48px;
  text-align: center;
  color: #f00;
}
.input-btn{
  display: flex;
  align-items: center;  
  justify-content: center;
  input{
    font-size: 24px;
    height: 40px;
  }      
  button{
    height: 40px;
    font-size: 24px;
    margin-left: 20px;
  }
}
.score{
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #eff;
  font-size: 38px;
  div{
    font-size: 18px;
  }
}
</style>
