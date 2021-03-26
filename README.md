# 贪吃蛇

#### 用Vue实现了一个贪吃蛇的小游戏，具体实现思路如下
1. 棋盘用一个50\*50的div实现，维护一个50*50的二维数组，循环得到棋盘，白色div为苹果，红色为蛇身，蓝色为蛇头
```
<div class="line" v-for="(v1, i1) in chess_map" :key="'o' + i1">
    <span :class="{'box': 1, 'active-box': v2 === 1, 's-head': v2 === 2, 'apple': v2 === 3}" v-for="(v2, i2) in v1" :key="'i' + i2">
    </span>
</div>
```
2. 维护一个贪吃蛇的身体各个位置的数组，此数组为二维，记录了贪吃蛇身体的各个位置的坐标
```
this.snake_pos_arr = [
                        [0, 0],
                        [0, 1],
                        [0, 2],
                        [0, 3],
                        [0, 4],
                    ];
```
3. 监听键盘输入，记录蛇下一步的移动方向
```
 mounted() {
    document.onkeydown = this.keydown
},
```
```
if (this.cur_direction === "left" && (e.key === "d" || e.key === "ArrowRight")) {
    return;
}
```
4. 当蛇移动时，尾巴位置对应的数组删除，并在数组末端增加一位代表新的头部位置，如果新的头部位置和苹果位置重合，就不删除尾巴
```
let newhead = [...this.snake_pos_arr[this.snake_pos_arr.length - 1]];
switch (this.new_direction) {
case "right":
    newhead[1] = newhead[1] + 1;
    break;
    ...
}
this.snake_pos_arr.push(newhead);
// 如果头的位置和苹果位置重合，就吃到了
if(this.apple_pos[0] !== newhead[0] || this.apple_pos[1] !== newhead[1]){
    this.snake_pos_arr.splice(0, 1);
}else{
    this.apple_pos = null
    this.score++
}
```
5. 判断死亡条件：头的位置超过棋盘即为撞墙，头的位置和身体位置重合即为碰撞到自己
```
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
```

6. 生成苹果，如果棋盘上没有苹果，那么在除过蛇身体所在部位的位置随机生成一个苹果
```
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
```