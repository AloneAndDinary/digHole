<template>
  <div class="home">
    <div>首页</div>
    <div id="potBottom">
      <div v-for="(item,index) in potBottom" :key="index">
        <p>{{item.type}}</p>
        <p>{{item.name}}</p>
      </div>
    </div>
    <div id="userList">
      <ul v-for="(item,index) in peopleList" :key="index">
        <li v-for="(card,i) in item.handCard" :key="i" :class="{isChecked:card.isChecked}" @click="selectCard(card,i)">
          <p>{{card.type}}</p>
          <p>{{card.name}}</p>
        </li>
      </ul>
    </div>
    <div id="potBottomList">
      <ul>
        <li v-for="(card,i) in potBottomList" :key="i">
          <p>{{card.type}}</p>
          <p>{{card.name}}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      typeList: ['梅花','方块','黑桃','红桃'], // 牌型列表
      nameList: ['A','2','3','4','5','6','7','8','9','10','J','Q','K'], // 牌面列表
      peopleList: [
        {
          handCard: [],
          isLandlord: false,
          number: 0,
          count: 0
        },
        {
          handCard: [],
          isLandlord: false,
          number: 0,
          count: 0
        },
        {
          handCard: [],
          isLandlord: false,
          number: 0,
          count: 0
        }
      ], // 玩家列表
      cardList: [], // 牌堆
      potBottom: [], //锅底
      gameStatus: 0, // 游戏状态  0 开始   1 叫分   2 出牌  3 结账
      potBottomList: [] // 锅底牌
    }
  },
  mounted() {
    this.generateCard();
  },
  methods: {
    // 生成
    generateCard() {
      this.nameList.forEach((name)=>{
        this.typeList.forEach((type,index)=> {
          if(name !== '3' || index !== 0) {
            this.cardList.push({
              name: name,
              type: type,
              isChecked: false,
              isSend: false,
              isUse: true
            });
          }
        });
      });
      this.removeNoUseCard();
    },
    // 移除不使用的牌
    removeNoUseCard() {
      this.cardList.forEach(card => {
        if(card.name === 'A' || card.name === '2') {
          card.isUse = false;
        }
      });
      this.licensingToPeople();
    },
    // 发牌给玩家
    licensingToPeople() {
      const sendCardList = JSON.parse(JSON.stringify(this.cardList.filter(item => item.isUse)));
      for(let i=0;i<13;i++){
        for(let j=0;j<this.peopleList.length;j++){
          let index = parseInt(this.randombetween(0,sendCardList.length-1));
          this.peopleList[j].handCard.push(sendCardList[index]);
          sendCardList.splice(index,1);
        }
      }
      this.potBottom = sendCardList;
    },
    // 取随机数
    randombetween(min, max){
      return min + (Math.random() * (max-min +1));
    },
    // 选择一张牌
    selectCard(card) {
      card.isChecked = !card.isChecked;
    },
    // 出牌
    sendCard() {
      this.getIsCheckedCard();
    },
    // 获取选中的牌
    getIsCheckedCard(user) {
      return user.handCard.filter(item => item.isChecked);
    },
    // 验证选中牌类型
    verifyCheckedCard() {}
  }
}
</script>
<style lang="less" scoped>
  ul{
    padding: 0;
    li{
      list-style: none;
    }
  }
  #potBottom{
    display: flex;
    div{
      width: 60px;
      margin: 0px 10px;
      text-align: center;
      border: 1px solid #ddd;
    }
  }
  #userList{
    ul{
      display: flex;
      flex-wrap: nowrap;
      li{
        width: 60px;
        margin: 0px 10px;
        text-align: center;
        border: 1px solid #ddd;
      }
      li.isChecked{
        border: 1px solid red;
      }
    }
  }
</style>
