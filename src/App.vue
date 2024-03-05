<template>
  <div class='firstContainer'>
    <div class="inp">
      <table class="tableInput1">
        <tr>
          <td rowspan="3">控制期</td>
          <td colspan="4">高峰</td>
          <td rowspan="3">超高峰系数</td>
        </tr>
        <tr>
          <td colspan="2">上行</td>
          <td colspan="2">上行</td>
        </tr>
        <tr>
          <td>上车</td>
          <td>下车</td>
          <td>上车</td>
          <td>下车</td>
        </tr>
        <tr>
          <td>初期</td>
          <td><input type="number" v-model.number="cal_table.aa"></td>
          <td><input type="number" v-model.number="cal_table.ab"></td>
          <td><input type="number" v-model.number="cal_table.ac"></td>
          <td><input type="number" v-model.number="cal_table.ad"></td>
          <td rowspan="3"><input type="number" step="0.01" v-model.number="k"></td>
        </tr>
        <tr>
          <td>近期</td>
          <td><input type="number" v-model.number="cal_table.ba"></td>
          <td><input type="number" v-model.number="cal_table.bb"></td>
          <td><input type="number" v-model.number="cal_table.bc"></td>
          <td><input type="number" v-model.number="cal_table.bd"></td>
        </tr>
        <tr>
          <td>远期</td>
          <td><input type="number" v-model.number="cal_table.ca"></td>
          <td><input type="number" v-model.number="cal_table.cb"></td>
          <td><input type="number" v-model.number="cal_table.cc"></td>
          <td><input type="number" v-model.number="cal_table.cd"></td>
        </tr>
      </table>

      <div class="divideLine"></div>

      <table class="tableInput2">
        <tr>
          <td><label>列车每节长度:</label></td>
          <td><input type="number" step="0.01" v-model.number="s_l"></td>
        </tr>
        <tr>
          <td><label>列车节数:</label></td>
          <td><input type="number" v-model.number="n_l"></td>
        </tr>
        <tr>
          <td><label>站台人流密度:</label></td>
          <td><input type="number" step="0.01" v-model.number="r_b"></td>
        </tr>
        <tr>
          <td><label>横向柱子个数:</label></td>
          <td><input type="number" v-model.number="n_b"></td>
        </tr>
        <tr>
          <td><label>柱宽:</label></td>
          <td><input type="number" step="0.01" v-model.number="c_b"></td>
        </tr>
      </table>

      <div class="divideLine"></div>

      <table class="tableInput3">
        <tr>
          <td><label>输入入口名称:</label></td>
          <td><input type="text" v-model.number="ename"></td>
        </tr>
        <tr>
          <td><label>输入入口人流量:</label></td>
          <td><input type="number" v-model.number="enumber"></td>
        </tr>
        <button @click="addPush">添加 +</button>
      </table>
      <ul>
        <li v-for="todo in ie" :key="todo.id">
          通道：{{ todo.text }}
          人数：{{ todo.num }}
          <button @click="removeTodo(todo)">删除</button>
        </li>
      </ul>
      <button @click="subData" class="postBtn">- admit (POST) - </button>
    </div>

    <div class="divider"></div>

    <div class="outp">
      <!-- <span>{{ mes }}</span> -->
      <div class="titleBox"># i1.站台有效长度数据计算</div>
      <div class="textBox" v-if="mes.len">{{ mes.len }}</div>
      <div class="titleBox"># i2.楼梯与自动楼梯</div>
      <div class="textBox" v-for="(str, index) in mes.stair" :key="index">{{ str }}</div>
      <div class="titleBox"># i3.站台宽度计算（客流计算法）</div>
      <div class="textBox" v-for="(str, index) in mes.width" :key="index">{{ str }}</div>
      <div class="titleBox"># i4.售检票设施数量计算</div>
      <div class="textBox" v-for="(str, index) in mes.equip" :key="index">{{ str }}</div>
      <div class="titleBox"># i5.出入口设计</div>
      <div v-for="(arr, index) in mes.enter" :key="index">
        <div class="textBox" v-for="(str, index) in arr" :key="index">{{ str }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

let id = 0

export default {
  data() {
    return {
      mes: {
        len: "",
        stair: [],
        width: [],
        equip: [],
        enter: []
      },
      cal_table:{
        aa: 401,
        ab: 558,
        ac: 740,
        ad: 503,
        ba: 487,
        bb: 724,
        bc: 900,
        bd: 636,
        ca: 591,
        cb: 870,
        cc: 1061,
        cd: 769,
        
      },
      s_l: 22.8,
      n_l: 4,
      r_b: 0.25,
      n_b: 2,
      c_b: 0.7,
      k: 1.18,
      ename: 'A1',
      enumber: 810,
      ie: [],
      enterk: [],
      enterv: []
    }
  },
  methods: {
    subData() {
      this.ie.forEach(item => {
        this.enterk.push(item.text);
        this.enterv.push(item.num);
      });
      const postData = {
        ckey: 'ys',
        s_l: this.s_l,
        n_l: this.n_l,
        qu_b: this.qu_b, 
        qd_b: this.qd_b, 
        r_b: this.r_b, 
        n_b: this.n_b, 
        c_b: this.c_b, 
        k: this.k,
        enterk: this.enterk,
        enterv: this.enterv
      };
      
      axios.post('http://127.0.0.1:8000/lea/', postData, {
      headers: {
        'Content-Type': 'application/json'
      }
    })
    .then(({data:res}) => {
      console.log(res)
      this.mes.len = res.len
      this.mes.stair = res.stair
      this.mes.width = res.width
      this.mes.equip = res.equip
      this.mes.enter = res.enter
      this.enterk = [],
      this.enterv = []
    })
    .catch(error => {
      console.error('Error', error)
    });
    },
    addPush() {
      this.ie.push({id:id++, text: this.ename, num: this.enumber})
      this.ename = '',
      this.enumber = 0
    },
    removeTodo(todo) {
      this.ie = this.ie.filter((t) => t !== todo)
    }
  },
  computed: {
    qu_b() {
      return parseInt(Math.max(
        this.cal_table.aa + this.cal_table.ac,
        this.cal_table.ba + this.cal_table.bc,
        this.cal_table.ca + this.cal_table.cc,
      )*this.k)
    },
    qd_b() {
      return parseInt(Math.max(
        this.cal_table.ab + this.cal_table.ad,
        this.cal_table.bb + this.cal_table.bd,
        this.cal_table.cb + this.cal_table.cd,
      )*this.k)
    },
  }
}
</script>
<style scoped>
.firstContainer {
  display: flex;
}

.inp {
  flex: 1;
  margin-right: 10px;
  background-color: white;
  border-radius: 5px;
  box-shadow:rgba(0, 0, 0, 0.45) 0px 0px 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 5px;
}

.outp {
  flex: 1;
  margin-left: 10px;
}
.divider {
  width: 1px;
  background-color: #efefef;
}
.tableInput1 input[type="number"] {
  width: 60px;
  border: none;
  outline: none;
  text-align: right;
  margin-left: -5px;
}
.tableInput1 {
  margin-top: 20px
}
.tableInput1 td {
  text-align: center;
}
.tableInput1 table {
  border-collapse: collapse;
  border-spacing: 0;
}
.tableInput1 td, .tableInput1 th {
  border: 1px solid black;
  padding: 8px;
}
/* .tableInput1{
  margin: 20px;
} */
/* .tableInput2 {
  padding-left: 125px;
  margin-bottom: 20px;
} */
.postBtn {
 width: 250px;
 height: 50px;
 margin: 20px;
 border-radius: 9px;
}
.divideLine {
  width:450px;
  height: 1px;
  background-color: #0000006b;
  margin: 20px;
}
.textBox {
  margin: 5px;
  padding: 5px;
  margin-bottom: 10px;
  background-color: white;
  border-radius: 5px;
  box-shadow:rgba(0, 0, 0, 0.45) 0px 0px 3px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}




</style>

