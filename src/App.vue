<template>
  <div class="app">
    <div class="word">
      <template v-if="!spell">
        <h1>{{ wordObj?.word }}</h1>
        <div class="voice">
          美音：/ {{ phonetic }} / ：
          <i class="iconfont icon-yuyin voice-i" :class="{ 'voice-color': voice }" @click="playVoice"></i>
        </div>
        <h3 v-show="show" class="meaning">{{ wordObj?.meaning }}</h3>
      </template>
      <div v-else class="spell-meaning">
        <h3>{{ wordObj?.meaning }}</h3>
        <div class="voice">
          语音：
          <i class="iconfont icon-yuyin voice-i" :class="{ 'voice-color': voice }" @click="playVoice"></i>
        </div>
      </div>

    </div>
    <div class="spell" v-show="spell">
      <input type="text" class="input" placeholder="请输入拼写的单词" v-model="spellWord">
    </div>
    <div class="btn">
      <template v-if="!spell">
        <button class="round btn-yes" @click="getWord">通过</button>
        <button class="round btn-no" @click="show = true">忘记</button>
      </template>
      <button v-else class="round btn-yes" @click="pass">提交</button>
    </div>
    <div class="count">
      <template v-if="!spell">
        <div>词组：{{ spellLength - tenWords.length + '/' + spellLength }}</div>
        <div>总数：{{ total - words.length + '/' + total }}</div>

      </template>
      <div v-else>拼写：{{ spellArrLength - spellArr.length + '/' + spellArrLength }}</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import res from './words.json'
import { Dialog, Toast } from 'vant';
import { jsonp } from 'vue-jsonp'
import CryptoJS from 'crypto-js';
// 随机十个单词的数组
type WordType = {
  word: string,
  meaning: string
}
const words = ref<WordType[]>([])
const total = res.length
const tenWords = ref<WordType[]>([])
const spellLength = ref(words.value.length < 10 ? words.value.length : 10)
const getTenWords = () => {
  if (words.value.length == 0) {
    words.value.push(...JSON.parse(JSON.stringify(res)))

  }
  const length = words.value.length < 10 ? words.value.length : 10
  // 获取要拼写的单词数
  spellLength.value = length
  for (let i = 0; i < length; i++) {
    let index = Math.floor(Math.random() * words.value.length)
    tenWords.value?.push(words.value[index])
  }
  spellArr.value = JSON.parse(JSON.stringify(tenWords.value))

}

let wordObj = ref<WordType>({
  word: '',
  meaning: ''
})
let show = ref<boolean>(false) // 翻译展示

// 拼写部分
const spell = ref(false)
const spellWord = ref('232')
const spellArr = ref<WordType[]>([])
const spellArrLength = ref(0)
const count = ref(0)
const getMeaning = () => {

  let index = Math.floor(Math.random() * spellArr.value.length)
  wordObj.value = spellArr.value[index]


  spellArr.value.splice(index, 1)
  console.log('count', count.value, 'length', spellArr.value.length);
  count.value--
  spellWord.value = ''
}
const pass = () => {
  if (count.value === 0) {
    spell.value = false
    Toast.success('再学一组');
    getTenWords()
    getWord()
    return
  }
  if (wordObj.value.word === spellWord.value) {
    getMeaning()
  } else {
    Toast.fail('拼写错误')
  }
}
const getWord = () => {

  if (tenWords.value.length == 0) {
    Dialog.confirm({
      title: '提示',
      message:
        '是否进入单词拼写？',
    })
      .then(() => {
        // on confirm
        console.log('确认了');
        spell.value = true
        spellArrLength.value = spellArr.value.length
        count.value = spellArrLength.value
        getMeaning()

      })
      .catch(() => {
        // on cancel
        console.log('取消了');
        getTenWords()
        getWord()

      });
    return
  }

  show.value = false
  let index = Math.floor(Math.random() * tenWords.value.length)
  wordObj.value = tenWords.value[index]
  getPhonetic()
  const res = tenWords.value.splice(index, 1)

  const wIndex = words.value.findIndex(item => item.word === wordObj.value?.word)
  words.value.splice(wIndex, 1)

}




// 语音
const voice = ref(false)
const player = new Audio();
const phonetic = ref('')
const getPhonetic = async () => {
  const appKey = '1d78e686733634b2';
  const key = '7vFD0HXrMKdnZZsOSw6BE4tYX9gJpgeX';//注意：暴露appSecret，有被盗用造成损失的风险
  const salt = (new Date).getTime();
  const curtime = Math.round(new Date().getTime() / 1000);
  const query = wordObj.value.word;
  console.log('1111', wordObj.value);

  // 多个query可以用\n连接  如 query='apple\norange\nbanana\npear'
  const from = 'en';
  const to = 'zh-CHS';
  const str1 = appKey + query + salt + curtime + key;
  const sign = CryptoJS.SHA256(str1).toString(CryptoJS.enc.Hex);
  const response = await jsonp('https://openapi.youdao.com/api', {
    q: query,
    appKey: appKey,
    salt: salt,
    from: from,
    to: to,
    sign: sign,
    signType: "v3",
    curtime: curtime,
  })
  phonetic.value = response.basic["us-phonetic"]
  console.log(response.basic["us-phonetic"]);
}
const playVoice = () => {
  console.log('点击了');
  player.src = 'http://dict.youdao.com/dictvoice?audio=' + wordObj.value.word
  voice.value = true
  player.play();
}

player.addEventListener('ended', () => {
  console.log('播放结束了');
  voice.value = false
})

getTenWords()
getWord()
</script>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.voice-color {
  color: #25bbfb;
}

.app {
  height: 100vh;
  overflow: hidden;
  background: url('@/assets/bg.jpg');
  background-size: cover;

  .word {

    width: 80%;
    height: 40%;
    text-align: center;
    margin: 2% auto;
    border: 2px solid #36b7f2;
    border-radius: 10px;
    padding: 2vh 2vw;

    h1 {
      margin: 2vh 0;

    }

    .voice {
      margin: 6vh;
      font-weight: bold;

      .voice-i {
        cursor: pointer;
        font-size: 18px;
      }
    }


    .spell-meaning {
      margin-top: 5vh;
    }
  }

  .spell {

    margin: 5vh auto;
    text-align: center;

    .input {
      width: 250px;
      height: 30px;
      border: 2px solid #36b7f2;
      border-radius: 100px;
      padding-left: 5px;
      text-align: center;
      outline: none;

      &::placeholder {
        color: #5ec6f7;
        font-size: 12px;
      }
    }
  }

  .btn {
    position: fixed;
    left: 0;
    bottom: 20%;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 5%;
    width: 100%;

    .round {
      border-radius: 20px;
      padding: 8px 23px;
      border: none;
      color: #fff;

      &:active {
        opacity: .8;
      }
    }

    .btn-yes {
      background-color: #409eff;
    }

    .btn-no {
      background-color: #f56c6c;
      margin-left: 10vh;
    }
  }

  .count {
    margin-top: 10vh;

    div {
      text-align: center;
    }
  }
}
</style>