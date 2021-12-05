<template>
  <div>
    <div v-if="!isLogin">
      <p>
        ユーザー名: <input v-model="userName" />
        <button @click="login">ログイン</button>
      </p>
    </div>
    <div v-else>
      <div>
        <p>ユーザー名: {{ userName }}</p>
        <button @click="logout">ログアウト</button>
      </div>

      <div>
        <p>
          送り先のユーザー名: <input v-model="receiveUser" /><button
            @click="openChat"
          >
            トーク画面を開く
          </button>
        </p>
      </div>
      <div v-if="isOpenChat">
        <p>メッセージ欄</p>
        <div class="msg-area">
          <div v-for="(message, index) in messages" :key="index">
            <MessageCard :text="message.text" :userName="message.sender" />
          </div>
        </div>
        <div>
          <input v-model="text" />
          <button @click="sendMessage(text, userName, receiveUser)">
            テキスト送信
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import MessageCard from "../components/MessageCard.vue";
import { reactive, toRefs } from "vue";
import { db } from "../plugins/firebase";
import {
  addDoc,
  orderBy,
  serverTimestamp,
  collection,
  query,
  where,
  onSnapshot,
  Unsubscribe,
} from "firebase/firestore";

interface IChatMsg {
  isDeleted: boolean;
  receiver: string;
  sender: string;
  text: string;
}

interface state {
  text: string;
  userName: string;
  isLogin: boolean;
  isOpenChat: boolean;
  messages: IChatMsg[];
  receiveUser: string;
  listener: Unsubscribe | null;
}

export default {
  components: {
    MessageCard,
  },
  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  setup() {
    const state = reactive<state>({
      text: "",
      userName: "",
      isLogin: false,
      isOpenChat: false,
      messages: [],
      receiveUser: "",
      listener: null,
    });

    const login = async () => {
      if (state.userName.length < 1) {
        return;
      }
      state.isLogin = true;
    };

    const logout = async () => {
      if (state.listener != null) {
        state.listener();
      }
      state.userName = "";
      state.isLogin = false;
      state.isOpenChat = false;
    };

    const openChat = async () => {
      if (state.receiveUser.length < 1) {
        return;
      }
      await observer();
      state.isOpenChat = true;
    };

    const observer = async () => {
      const q = query(
        collection(db, "message"),
        where("isDeleted", "==", false),
        orderBy("createdAt")
      );
      state.listener = onSnapshot(q, (snapshot) => {
        snapshot.docChanges().forEach((change) => {
          if (change.type === "added") {
            const msg = change.doc.data() as IChatMsg;
            if((msg.receiver == state.userName && msg.sender == state.receiveUser) || (msg.receiver == state.receiveUser && msg.sender == state.userName)){
              state.messages.push(change.doc.data() as IChatMsg);
            }
          }
        });
      });
    };

    const sendMessage = async (
      text: string,
      sender: string,
      receiver: string
    ) => {
      const citiesDoc = collection(db, "message");
      await addDoc(citiesDoc, {
        text: text,
        sender: sender,
        receiver: receiver,
        createdAt: serverTimestamp(),
        isDeleted: false,
      });
      state.text = "";
    };

    return {
      ...toRefs(state),
      login,
      logout,
      openChat,
      observer,
      sendMessage,
    };
  },
};
</script>
<style scoped>
.msg-area {
  width: 400px;
  margin: 0 auto;
  background-color: grey;
}
.myMsg {
  text-align: right;
}
.otherMsg {
  text-align: left;
}
</style>