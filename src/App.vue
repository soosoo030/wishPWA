<template>
  <v-app>
    <v-app-bar app color="#FAE8A8">
      <v-toolbar-title>WishMallow</v-toolbar-title>
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
          <v-col cols="2" my-2>
            <v-btn
              fab
              max-height="50px"
              max-width="50px"
              color="#FEA14B"
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
            <v-card
              flat
              color="grey lighten-3"
              v-if="!item.b_edit"
            >
              <v-card-title>
                <h2>{{ item.wish_title }}</h2>
              </v-card-title>
              <v-card-text>
                <p>목표 금액 : {{ item.wish_goalMoney }}</p>
                <p>현재 금액 : {{ item.wish_currMoney }}</p>
                <div align="right">
                  <div>
                    <v-text-field label="추가할 금액을 입력해주세요" clearable v-if="item.b_Flag==true" v-model="iInputMoney"></v-text-field>
                  </div>
                  <!-- 목표금액과 현재금액이 일치하거나 이상일때 생기는 celebration 아이콘 -->
                  <v-icon class="pointer" v-if="item.wish_currMoney >= item.wish_goalMoney" @click="fnCompleteWish(item)">celebration</v-icon>
                  <!-- 현재 금액에 금액 추가할 수 있는 버튼 -->
                  <v-icon class="pointer" @click="fnSaveMoney(item)"
                    >savings</v-icon
                  >
                  <v-icon class="pointer" @click="fnSetEditWish(item['.key'])"
                    >create</v-icon
                  >
                  <v-icon class="pointer" @click="fnRemoveWish(item['.key'])"
                    >delete</v-icon
                  >
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
                  >save</v-icon>
                <v-icon class="pointer" @click="fnCancelEdit(item['.key'])">cancel</v-icon>
              </v-card-actions>
            </v-card>
          
          </v-col>
        </v-row>
      </v-container>
      <v-container>
        <v-col cols="12">
          <div class="text-center" width="100%">명예의 전당</div>
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
              flat
              color="black"
              v-if="item.b_completed"
              dark
            >
              <v-card-title>
                <h2>{{ item.wish_title }}</h2>
              </v-card-title>
              <v-card-text>
                <p>{{ item.wish_goalMoney }}원 모았어요(❁´◡`❁)</p>
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
      bFlag:false,
      iInputMoney:0,
    };
  },
  // 파이어베이스를 쉽게 사용하도록 wishs 변수로 변경
  firebase: {
    wishs: oWishsinDB,
  },
  methods: {
    // title, money, 완료, 상탯값을 DB에 저장
    fnSubmitWish() {
      oWishsinDB.push({
        wish_title: this.sWishTitle,
        wish_goalMoney: this.sWishGoalMoney,
        wish_currMoney: this.sWishCurrMoney,
        b_completed: false,
        b_edit: false,
        b_Flag: false,
      });
      this.sWishTitle = "";
      this.sWishGoalMoney = "";
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
    fnSaveMoney(pItem){
      const sKey = pItem['.key'];
      if(pItem.b_Flag==true){
        // 입력된 금액을 더해서 currMoney 값을 변경해주기
        console.log(this.iInputMoney);
        oWishsinDB.child(sKey).update({
          b_Flag:false,
          wish_currMoney : parseInt(pItem.wish_currMoney)+parseInt(this.iInputMoney),
        })
      }else{
        oWishsinDB.child(sKey).update({
        b_Flag:true,
      })
      }
      this.iInputMoney = 0;
    },

    //전달된 wish의 b_edit을 읽기모드로 변경
    fnCancelEdit(pKey) {
      oWishsinDB.child(pKey).update({
        b_edit: false,
      });
    },
    //전달된 wish의 수정값을 DB에 저장
    fnSaveEdit(pItem) {
      const sKey = pItem[".key"];
      oWishsinDB.child(sKey).set({
        wish_title: pItem.wish_title,
        wish_goalMoney: pItem.wish_goalMoney,
        wish_currMoney: pItem.wish_currMoney,
        b_completed: pItem.b_completed,
        b_edit: false,
        b_Flag: false,
      });
    },
    //완료된 wish의 b_completed 변경값 저장
    fnCompleteWish(pItem){
      const sKey = pItem['.key'];
      oWishsinDB.child(sKey).update({
        b_completed:true
      })
    }
  },

  computed:{
    completedWishs: function(){
      return this.wishs.filter(function (wish){
        return wish.b_completed
      })
    },
    uncompletedWishs: function(){
      return this.wishs.filter(function(wish){
        return !wish.b_completed
      })
    }
  }
};
</script>
<style>
.pointer{
  cursor:pointer;
}
.style_completed{
  
}
</style>