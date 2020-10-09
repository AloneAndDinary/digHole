<template>
<div class="home">
    <div id="controlList">
        <span @click="gameStart">开始</span>
    </div>
    <template v-if="gameStatus >= 2">
        <div id="record">
            <span v-for="(item, index) in record" :key="index">{{ item }}</span>
        </div>
    </template>
    <template v-if="gameStatus >= 1">
        <div id="potBottom">
            <h3>锅底</h3>
            <div v-for="(item, index) in potBottom" :key="index">
                <p>{{ item.type }}</p>
                <p>{{ item.name }}</p>
            </div>
        </div>
        <div id="userList">
            <h3>玩家列表</h3>
            <ul v-for="(item, index) in peopleList" :key="index">
                <div class="btnList">
                    <button @click="sendCard">出牌</button>
                    <button>过</button>
                </div>
                <li v-for="(card, i) in item.handCard" :key="i" :class="{ isChecked: card.isChecked }" @click="selectCard(card, item)">
                    <p>{{ card.type }}</p>
                    <p>{{ card.name }}</p>
                </li>
            </ul>
        </div>
        <div id="potBottomList">
            <h3>锅底牌堆</h3>
            <ul>
                <li v-for="(card, i) in potBottomList" :key="i">
                    <p>{{ card.type }}</p>
                    <p>{{ card.name }}</p>
                </li>
            </ul>
        </div>
    </template>
</div>
</template>

<script>
export default {
    name: "Home",
    data() {
        return {
            typeList: ["梅花", "方块", "黑桃", "红桃"], // 牌型列表
            nameList: [{
                    name: "A",
                    value: 1
                },
                {
                    name: "2",
                    value: 2
                },
                {
                    name: "3",
                    value: 3
                },
                {
                    name: "4",
                    value: 4
                },
                {
                    name: "5",
                    value: 5
                },
                {
                    name: "6",
                    value: 6
                },
                {
                    name: "7",
                    value: 7
                },
                {
                    name: "8",
                    value: 8
                },
                {
                    name: "9",
                    value: 9
                },
                {
                    name: "10",
                    value: 10
                },
                {
                    name: "J",
                    value: 11
                },
                {
                    name: "Q",
                    value: 12
                },
                {
                    name: "K",
                    value: 13
                }
            ], // 牌面列表
            peopleList: [{
                    id: 0,
                    handCard: [],
                    isLandlord: false,
                    number: 0,
                    count: 0,
                    isActive: false
                },
                {
                    id: 1,
                    handCard: [],
                    isLandlord: false,
                    number: 0,
                    count: 0,
                    isActive: false
                },
                {
                    id: 2,
                    handCard: [],
                    isLandlord: false,
                    number: 0,
                    count: 0,
                    isActive: false
                }
            ], // 玩家列表
            cardList: [], // 牌堆
            potBottom: [], //锅底
            gameStatus: -1, // 游戏状态  0 开始   1 叫分   2 出牌  3 结账
            record: [1, 2, 3], // 可叫分数列表
            potBottomList: [], // 锅底牌
            redThreeCard: {
                name: "4",
                type: "红桃",
                value: 4,
                isChecked: false,
                isSend: false,
                isUse: true
            },
            currentPeople: [], // 当前出牌人
            cardType: "", // 'one','two','mostThree' 牌型
            currentRound: 0, // 当前轮状态
            prevSendCard: []
        };
    },
    watch: {
        gameStatus(val) {
            switch (val) {
                case 0:
                    this.generateCard();
                    setTimeout(() => {
                        this.gameStatus = 1;
                    }, 500);
                    break;
                case 1:
                    this.licensingToPeople();
                    setTimeout(() => {
                        this.gameStatus = 2;
                    }, 500);
                    break;
                case 2:
                    this.findCardToPeople(this.redThreeCard);
                    this.currentPeople.isActive = true;
                    console.log(this.peopleList);
            }
        }
    },
    mounted() {},
    methods: {
        // 开始游戏
        gameStart() {
            this.gameStatus = 0;
        },
        // 生成
        generateCard() {
            this.nameList.forEach(name => {
                this.typeList.forEach((type, index) => {
                    if (name.name !== "3" || index !== 0) {
                        this.cardList.push({
                            name: name.name,
                            value: name.value,
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
                if (card.name === "A" || card.name === "2") {
                    card.isUse = false;
                }
            });
        },
        // 发牌给玩家
        licensingToPeople() {
            const sendCardList = JSON.parse(
                JSON.stringify(this.cardList.filter(item => item.isUse))
            );
            for (let i = 0; i < 13; i++) {
                for (let j = 0; j < this.peopleList.length; j++) {
                    let index = parseInt(this.randombetween(0, sendCardList.length - 1));
                    this.peopleList[j].handCard.push(sendCardList[index]);
                    sendCardList.splice(index, 1);
                }
            }
            this.peopleList.forEach(item=>{
                item.handCard.sort((a,b)=>{
                    return a.value-b.value;
                })
            });
            sendCardList.sort((a,b)=>{
                return a.value-b.value;
            });
            this.potBottom = sendCardList;
        },
        // 取随机数
        randombetween(min, max) {
            return min + Math.random() * (max - min + 1);
        },
        // 选择一张牌
        selectCard(card, item) {
            if (item.isActive) {
                card.isChecked = !card.isChecked;
            }
        },
        // 出牌
        sendCard() {
            const selectCard = this.getIsCheckedCard(this.currentPeople);
            if (this.verifyCheckedCard(selectCard)) {
                selectCard.forEach(card=>{
                    this.currentPeople.handCard.forEach((item, index)=>{
                        if(card.value === item.value && card.type===item.type){
                            this.currentPeople.handCard.splice(index,1);
                        }
                    })
                });
                this.potBottomList.push(...selectCard);
            } else {
                alert('请选择正确的牌型');
            }
        },
        // 获取选中的牌
        getIsCheckedCard(user) {
            return user.handCard.filter(item => item.isChecked);
        },
        // 查找一张牌所属人
        findCardToPeople(card) {
            this.peopleList.forEach(item => {
                const findCard = item.handCard.filter(
                    detail => detail.type === card.type && detail.name === card.name
                );
                if (findCard.length > 0) {
                    this.currentPeople = item;
                }
            });
        },
        // 验证选中牌类型是否符合规则
        verifyCheckedCard(verifyCardList) {
            let flag = false;
            let cardNumberList = [];
            verifyCardList.forEach(item => {
                cardNumberList.push(item.value);
            });
            cardNumberList = cardNumberList.sort((a, b) => a - b);
            console.log(cardNumberList);
            let sendCardType = "";
            switch (cardNumberList.length) {
                case 1:
                    sendCardType = "one";
                    break;
                case 2:
                    sendCardType = "two";
                    break;
                default:
                    sendCardType = "mostThree";
                    break;
            }
            this.cardType = sendCardType;
            if (sendCardType === "one") {
                flag = true;
            } else if (sendCardType === "two") {
                if (cardNumberList[0] === cardNumberList[1]) {
                    flag = true;
                } else {
                    flag = false;
                }
            } else {
                let min = Math.min(...cardNumberList);
                cardNumberList.forEach((item, index) => {
                    if (index > 0 && item - min === 1) {
                        flag = true;
                    }
                });
                console.log(flag);
            }
            return flag;
        }
    }
};
</script>

<style lang="less" scoped>
ul {
    padding: 0;

    li {
        list-style: none;
    }
}

#controlList {
    display: flex;

    span {
        width: 60px;
        border: 1px solid #ddd;
        -webkit-border-radius: 20px;
        -moz-border-radius: 20px;
        border-radius: 20px;
        margin: 30px 10px;
        text-align: center;
        cursor: pointer;
    }
}

#record {
    display: flex;

    span {
        width: 60px;
        border: 1px solid #ddd;
        -webkit-border-radius: 20px;
        -moz-border-radius: 20px;
        border-radius: 20px;
        margin: 30px 10px;
        text-align: center;
        cursor: pointer;
    }
}

#potBottom {
    display: flex;

    div {
        width: 60px;
        margin: 0px 10px;
        text-align: center;
        border: 1px solid #ddd;
    }
}

#userList {
    ul {
        display: flex;
        flex-wrap: nowrap;

        li {
            width: 60px;
            margin: 0px 10px;
            text-align: center;
            border: 1px solid #ddd;
        }

        li.isChecked {
            border: 1px solid red;
        }
    }
}
#potBottomList {
    ul {
        display: flex;
        flex-wrap: nowrap;
        li {
            width: 60px;
            margin: 0px 10px;
            text-align: center;
            border: 1px solid #ddd;
        }
    }
}
</style>
