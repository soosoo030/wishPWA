<template>
  <v-app>
    <v-app-bar app color="#FAE8A8">
      <v-toolbar-title>✨WishMallow</v-toolbar-title>
    </v-app-bar>
    <v-main>
      <v-container>
        <v-row>
          <v-col cols="8" offset="1">
            <v-text-field
              label="title"
              autofocus
              v-model="sWishTitle"
            ></v-text-field>
            <v-text-field
              label="목표 금액"
              v-model="sWishGoalMoney"
            ></v-text-field>
          </v-col>
          <v-col cols="2" my-2 class="submitBtnDiv">
            <v-btn
              height="80%"
              width="80%"
              color="#b50d0c"
              dark
              @click="fnSubmitWish()"
            >
              <v-icon>add_shopping_cart</v-icon>
            </v-btn>
          </v-col>
          <v-col
            cols="12"
            sm="6"
            md="4"
            lg="3"
            xl="2"
            v-for="item in uncompletedWishs"
            :key="item.key"
          >
            <!-- 기본 상태 -->
            <v-card flat color="#FAE8A8" v-if="!item.b_edit">
              <v-card-title>
                <h2>{{ item.wish_title }}</h2>
                <v-spacer></v-spacer>
                <!-- 목표금액과 현재금액이 일치하거나 이상일때 생기는 celebration 아이콘 -->
                <v-icon
                  class="pointer"
                  v-if="item.wish_currMoney >= item.wish_goalMoney"
                  @click="fnCompleteWish(item)"
                  color="#b50d0c"
                  >celebration</v-icon
                >
                <v-icon
                  :class="{ style_iconActive: item.b_historyFlag }"
                  @click="fnSetHistoryFlag(item)"
                  >receipt_long</v-icon
                >
              </v-card-title>
              <v-card-text>
                <p>목표 금액 : {{ item.wish_goalMoney }}</p>
                <p>현재 금액 : {{ item.wish_currMoney }}</p>

                <!-- 입금 내역 -->
                <div
                  v-for="historyItem in item.historyArr"
                  v-if="item.b_historyFlag"
                >
                  • {{ historyItem.date | yyyyMMdd }} :
                  {{ historyItem.money }}원
                </div>

                <div align="right">
                  <div>
                    <v-text-field
                      label="추가할 금액을 입력해주세요"
                      clearable
                      v-if="item.b_Flag == true"
                      v-model="iInputMoney"
                    ></v-text-field>
                  </div>
                  <div v-if="item.b_Flag">
                    <v-icon
                      :class="{ style_iconActive: item.b_Flag }"
                      @click="fnSaveMoney(item)"
                      >savings</v-icon
                    >
                    <v-icon
                      class="pointer ml-2"
                      @click="fnCancelSave(item['.key'])"
                      >cancel</v-icon
                    >
                  </div>
                  <!-- 현재 금액에 금액 추가할 수 있는 버튼 -->
                  <div v-else>
                    <v-icon class="pointer" @click="fnSaveMoney(item)"
                      >savings</v-icon
                    >
                    <v-icon class="pointer" @click="fnSetEditWish(item['.key'])"
                      >create</v-icon
                    >
                    <v-icon class="pointer" @click="DialogDelete(item['.key'])"
                      >delete</v-icon
                    >
                  </div>
                </div>
              </v-card-text>
            </v-card>

            <!-- edit 가능 상태 -->
            <v-card v-else dark>
              <v-card-text>
                <v-text-field
                  label="title"
                  autofocus
                  clearable
                  v-model="item.wish_title"
                ></v-text-field>
              </v-card-text>
              <v-card-text>
                <v-text-field
                  label="목표 금액"
                  clearable
                  v-model="item.wish_goalMoney"
                ></v-text-field>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-icon class="pointer mr-2" @click="fnSaveEdit(item)"
                  >save</v-icon
                >
                <v-icon class="pointer" @click="fnCancelEdit(item['.key'])"
                  >cancel</v-icon
                >
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>

      <v-container>
        <v-col cols="12">
          <h1 class="text-center" width="100%">명예의 전당</h1>
        </v-col>
        <v-row>
          <v-col
            cols="12"
            sm="6"
            md="4"
            lg="3"
            xl="2"
            v-for="item in completedWishs"
            :key="item.key"
          >
            <!-- 완료 상태 -->
            <v-card
              class="style_completed"
              flat
              color="#345079"
              v-if="item.b_completed"
              dark
            >
              <v-card-title>
                <h2>{{ item.wish_title }}</h2>
                <v-spacer></v-spacer>
                <v-icon
                  :class="{ style_iconActive: item.b_historyFlag }"
                  @click="fnSetHistoryFlag(item)"
                  >receipt_long</v-icon
                >
                <v-icon class="pointer" @click="DialogDelete(item['.key'])"
                  >delete</v-icon
                >
              </v-card-title>
              <v-card-text>
                <p>{{ item.wish_goalMoney }}원 모았어요(❁´◡`❁)</p>
                <!-- 입금 내역 -->
                <div
                  v-for="historyItem in item.historyArr"
                  v-if="item.b_historyFlag"
                >
                  • {{ historyItem.date | yyyyMMdd }} :
                  {{ historyItem.money }}원
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
      <v-footer fixed dark>
        <div class="mx-auto">CREATED BY soosoo030</div>
      </v-footer>
    </v-main>
  </v-app>
</template>

<script>
// 파이어베이스 DB 가져옴
import { oWishsinDB } from "@/datasources/firebase";

export default {
  name: "App",

  data() {
    return {
      wishs: [],
      sWishTitle: "",
      sWishGoalMoney: "",
      sWishCurrMoney: 0,
      bFlag: false,
      iInputMoney: 0,
      //입금내역_날짜, 입금내역_금액
      historyArr: [],
      b_historyFlag: false,
    };
  },
  // 파이어베이스를 쉽게 사용하도록 wishs 변수로 변경
  firebase: {
    wishs: oWishsinDB,
  },
  methods: {
    // title, money, 완료, 상탯값을 DB에 저장
    fnSubmitWish() {
      if(this.Validation(parseInt(this.sWishGoalMoney))){
      oWishsinDB.push({
        wish_title: this.sWishTitle,
        wish_goalMoney: this.sWishGoalMoney,
        wish_currMoney: this.sWishCurrMoney,
        b_completed: false,
        b_edit: false,
        b_Flag: false,
        b_historyFlag: false,
        historyArr: [],
      });
      this.sWishTitle = "";
      this.sWishGoalMoney = "";
    }else{ //유효성 검사 실패 시
      this.$dialog.warning({title:'금액 오류', text:'올바른 금액을 입력해주세요 ●˙^˙●'});
    }
    },
    // 전달된 wish를 DB에서 삭제
    fnRemoveWish(pKey) {
      oWishsinDB.child(pKey).remove();
    },
    //전달된 wish의 b_edit을 수정모드로 변경
    fnSetEditWish(pKey) {
      oWishsinDB.child(pKey).update({
        b_edit: true,
      });
    },
    //전달된 금액을 해당 wish의 currMoney에 추가하기
    fnSaveMoney(pItem) {
      const sKey = pItem[".key"];
      if (pItem.b_Flag == true) {
        if(this.Validation(parseInt(this.iInputMoney))){
        // 입력된 금액을 더해서 currMoney 값을 변경해주기
        if (pItem.historyArr) {
          oWishsinDB.child(sKey).update({
            b_Flag: false,
            wish_currMoney:
              parseInt(pItem.wish_currMoney) + parseInt(this.iInputMoney),
            // 내역 추가
            historyArr: [
              ...pItem.historyArr,
              { date: Date.now(), money: parseInt(this.iInputMoney) },
            ],
          });
        } else {
          oWishsinDB.child(sKey).update({
            b_Flag: false,
            wish_currMoney:
              parseInt(pItem.wish_currMoney) + parseInt(this.iInputMoney),
            // 내역 추가
            historyArr: [
              { date: Date.now(), money: parseInt(this.iInputMoney) },
            ],
          });
        }
      }else{
        this.$dialog.warning({title:'금액 오류', text:'올바른 금액을 입력해주세요 ●˙^˙●'});
      }
        
        // 목표 달성 시, 다이얼로그 띄우기
        if (
          pItem.wish_currMoney + parseInt(this.iInputMoney) >=
          pItem.wish_goalMoney
        ) {
          this.DialogCelebration(pItem);
        }
      } else {
        oWishsinDB.child(sKey).update({
          b_Flag: true,
        });
      }
      this.iInputMoney = 0;
    },
    // 전달된 wish의 금액 입력 상태 취소하기
    fnCancelSave(pKey){
      oWishsinDB.child(pKey).update({ b_Flag: false, })
    },

    //전달된 wish의 b_edit을 읽기모드로 변경
    fnCancelEdit(pKey) {
      oWishsinDB.child(pKey).update({ b_edit: false, });
    },
    //전달된 wish의 수정값을 DB에 저장
    fnSaveEdit(pItem) {
      const sKey = pItem[".key"];
      if(this.Validation(parseInt(pItem.wish_goalMoney))){
        oWishsinDB.child(sKey).set({
        wish_title: pItem.wish_title,
        wish_goalMoney: pItem.wish_goalMoney,
        wish_currMoney: pItem.wish_currMoney,
        b_completed: pItem.b_completed,
        b_edit: false,
        b_Flag: false,
      });
      }else{
        this.$dialog.warning({title:'금액 오류', text:'올바른 금액을 입력해주세요 ●˙^˙●'});
      }
    },
    //완료된 wish의 b_completed 변경값 저장
    fnCompleteWish(pItem) {
      const sKey = pItem[".key"];
      oWishsinDB.child(sKey).update({
        b_completed: true,
      });
    },
    fnSetHistoryFlag(pItem) {
      const sKey = pItem[".key"];
      oWishsinDB.child(sKey).update({
        b_historyFlag: !pItem.b_historyFlag,
      });
    },

    // 삭제 전 확인용 팝업
    DialogDelete: async function (pKey) {
      const r = await this.$dialog.warning({
        title: "정말 삭제하시겠습니까?",
        text: "삭제 시, 되돌릴 수 없습니다.",
      });
      if (!r) return false;
      else if (r) return this.fnRemoveWish(pKey);
    },
    // 완료 시 축하용 팝업
    DialogCelebration: async function (item) {
      const r = await this.$dialog.confirm({
        title: "목표금액을 달성했습니다! (*ˊᵕˋo💐o",
        text: "(ง˙∇˙)ว  명예의 전당으로 이동할까요?",
      });
      if (r) return this.fnCompleteWish(item);
    },
    // 유효성 검사 함수
    Validation : function(num){
      // (1) 숫자 형태 여야 하고
      // (2) 0이면 안 됨
      if(Number.isInteger(num) && num !== 0){
        return true;
      }else{ return false;}
    },
  },

  computed: {
    completedWishs: function () {
      return this.wishs.filter(function (wish) {
        return wish.b_completed;
      });
    },
    uncompletedWishs: function () {
      return this.wishs.filter(function (wish) {
        return !wish.b_completed;
      });
    },
  },
  filters: {
    yyyyMMdd: function (value) {
      if (value == "") return "";
      var js_date = new Date(value);

      var year = js_date.getFullYear();
      var month = js_date.getMonth() + 1;
      var day = js_date.getDate();

      if (month < 10) {
        month = "0" + month;
      }
      if (day < 10) {
        day = "0" + day;
      }
      return year + "년 " + month + "월 " + day + "일";
    },
  },
};
</script>
<style>
.pointer {
  cursor: pointer;
}

.submitBtnDiv {
  display: flex;
  justify-content: center;
  align-items: center;
}
.style_iconActive {
  color: #b50d0c !important;
}
</style>
