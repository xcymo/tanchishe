<template>
  <div class="hello">
    <div class="game-over" v-if="game_over">
      <div>游戏结束!</div>
      <div>最终得分：{{score}}</div>
      <div><button @click="init">再来一局</button></div>
    </div>
    <div class="score" v-if="!game_over">{{score}}</div>
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
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      cur_direction: "right",
      new_direction: "right",
      chess_map: [],
      pos_arr: [],
      game_over: false,
      counter: "",
      speed: 100,
      apple_pos: null,
      score: 0,
    };
  },
  mounted() {
    document.onkeydown = this.keydown
    this.init();
  },
  methods: {
    init() {
      this.game_over = false
      this.cur_direction = "right"
      this.new_direction = "right"
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
      this.pos_arr = [
        [0, 0],
        [0, 1],
        [0, 2],
        [0, 3],
        [0, 4],
      ];
      this.counter = setTimeout(() => {
        this.go();        
      }, this.speed);
    },
    go() {
      // 生成苹果
      this.genApple()

      // 计算头部位置
      let newhead = [...this.pos_arr[this.pos_arr.length - 1]];
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
      // 重置当前位置
      this.pos_arr.push(newhead);
      if(this.apple_pos[0] !== newhead[0] || this.apple_pos[1] !== newhead[1]){
        this.pos_arr.splice(0, 1);
      }else{
        this.apple_pos = null
        this.score++
      }

      // 改变当前状态的方向
      this.cur_direction = this.new_direction;

      // 判断游戏是否结束
      if (this.checkGameEnd(newhead)) {
        this.game_over = true
        clearTimeout(this.counter)
        return;
      }

      // 清空地图并重新填
      this.chess_map.fill([]);
      this.chess_map = this.chess_map.map(() => {
        let arr = new Array(50);
        arr.fill(0);
        return arr;
      });
      this.pos_arr.forEach((v, i) => {
        if(i === this.pos_arr.length - 1){
          this.chess_map[v[0]][v[1]] = 2;
        } else{
          this.chess_map[v[0]][v[1]] = 1;
        }
      });
      if(this.apple_pos){
        this.chess_map[this.apple_pos[0]][this.apple_pos[1]] = 3
      }
      this.counter = setTimeout(() => {
        this.go();
      }, this.speed);
    },
    // 检查游戏是否结束
    checkGameEnd(newhead) {
      let res = false;
      // 判断是否碰墙
      if (newhead[0] >= 50 || newhead[1] >= 50 || newhead[0] < 0 || newhead[1] < 0) {
        res = true;
      }
      // 判断是否碰自己
      for(let i = 0; i < this.pos_arr.length - 2; i++){
        if(this.pos_arr[i][0] === newhead[0] && this.pos_arr[i][1] === newhead[1]){
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
.score{
  text-align: right;
  color: #00f;
  font-size: 38px;
}
</style>
