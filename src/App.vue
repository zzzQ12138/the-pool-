<template>
  <header>
    <div class="InfBox" @click="ShowInf">说明</div>
    <div class="AddBox" @click="ShowAdd">添加</div>
    <div class="RanBox" @click="RanWord">随机</div>
    <div class="CouBox" @click="CouWord">{{ buttonText }}</div>
    <div class="DelBox" @click="DelWord">删除</div>
    <div class="CleBox" @click="CleWord">清空</div>
  </header>

  <main>
    <div class="add" v-show="showadd">
      <input type="text" class="TextInput" placeholder="输入标题" v-model="TextInput">
      <textarea class="AreaInput" placeholder="输入内容" v-model="AreaInput"></textarea>
      <button class="SubBtn" @click="AddWord">添加</button>
      <span class="CloBtn" @click="CloAdd"> &times; </span>
    </div>
  </main>
  <div class="mod" v-show="showMod">
    <div class="ModCon">
      <p>{{ modText }}</p>
    </div>
    <span class="CloBtn" @click="closeMod"> &times; </span>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const showadd = ref(false);
const TextInput = ref('');
const AreaInput = ref('');
const showMod = ref(false);
const modText = ref('');
const randomCount = ref(5);
const isRandomMode = ref(false);
const buttonText = ref("数量");

let WordDatas = [];
const WordSpaces = [];
let ddl = false;

// 说明功能
const ShowInf = () => {
  alert(`这是一个记忆辅助网页：
  看见词条时尝试回忆相关知识，不确定时点击查看详情。
  可通过'添加'、'删除'和'清空'功能管理词条。
  '随机'功能可随机显示部分词条，'数量'功能可调整显示数量或重排词条。`);
};

const closeMod = () => {
  showMod.value = false;
};

// 添加表单的切换
const ShowAdd = () => {
  showadd.value = !showadd.value;
};

const CloAdd = () => {
  showadd.value = false;
};

// 智能排列词条（确保不重叠）
const AllWordPlace = () => {
  const main = document.querySelector('main');
  if (!main || WordSpaces.length === 0) return;
  
  const placed = [];
  // 按面积从大到小排序（大词条优先放置）
  const sortedWords = [...WordSpaces].sort((a, b) => 
    (b.offsetWidth * b.offsetHeight) - (a.offsetWidth * a.offsetHeight)
  );

  sortedWords.forEach(wordDom => {
    const wordwidth = wordDom.offsetWidth;
    const wordheight = wordDom.offsetHeight;
    let x, y, tryCount = 0, overlap;
    
    do {
      overlap = false;
      x = Math.random() * (main.clientWidth - wordwidth);
      y = Math.random() * (main.clientHeight - wordheight - 50);
      
      for (const old of placed) {
        if (x + wordwidth > old.x &&
            x < old.x + old.width &&
            y + wordheight > old.y &&
            y < old.y + old.height) {
          overlap = true;
          break;
        }
      }
      tryCount++;
    } while (overlap && tryCount < 100);
    
    Object.assign(wordDom.style, { 
      left: `${x}px`, 
      top: `${y}px`,
      display: wordDom.style.display // 保持原有显示状态
    });
    
    placed.push({ x, y, width: wordwidth, height: wordheight });
  });
};

// 添加词条
const AddWord = () => {
  const text = TextInput.value.trim();
  const area = AreaInput.value.trim();
  if (text && area) {
    const main = document.querySelector('main');
    const word = document.createElement('div');
    word.className = 'word';
    word.textContent = text;
    main.appendChild(word);
    
    WordSpaces.push(word);
    WordDatas.push({ text, area });
    
    // 设置初始随机位置
    const wordwidth = word.offsetWidth;
    const wordheight = word.offsetHeight;
    const x = Math.random() * (main.clientWidth - wordwidth);
    const y = Math.random() * (main.clientHeight - wordheight - 50);
    Object.assign(word.style, { left: `${x}px`, top: `${y}px` });
    
    word.addEventListener('click', () => {
      modText.value = area;
      showMod.value = true;
    });
    
    TextInput.value = '';
    AreaInput.value = '';
    showadd.value = false;
  }
};

// 删除功能
const DelWord = () => {
  ddl = !ddl;
  WordSpaces.forEach((wordDom, idx) => {
    let CloBtn = wordDom.querySelector('.CloBtn');
    if (CloBtn) wordDom.removeChild(CloBtn);
    if (ddl) {
      CloBtn = document.createElement('span');
      CloBtn.className = 'CloBtn';
      CloBtn.innerHTML = '&times;';
      CloBtn.onclick = (e) => {
        e.stopPropagation();
        wordDom.remove();
        WordSpaces.splice(idx, 1);
        WordDatas.splice(idx, 1);
      };
      wordDom.appendChild(CloBtn);
    }
  });
};

// 清空功能
const CleWord = () => {
  WordSpaces.forEach(wordDom => {
    wordDom.remove();
  });
  WordDatas = [];
  WordSpaces.length = 0;
};

// 随机/全部切换
const RanWord = () => {
  if (!isRandomMode.value) {
    // 进入随机模式
    AllWordPlace(); // 先重排所有词条
    const total = WordSpaces.length;
    if (total === 0) return;
    
    // 随机选择要显示的词条
    const showIdx = Array.from({ length: total }, (_, i) => i)
      .sort(() => 0.5 - Math.random())
      .slice(0, Math.min(randomCount.value, total));
    
    WordSpaces.forEach((wordDom, idx) => {
      wordDom.style.display = showIdx.includes(idx) ? '' : 'none';
    });
    
    isRandomMode.value = true;
    buttonText.value = "重排";
  } else {
    // 退出随机模式
    WordSpaces.forEach(wordDom => {
      wordDom.style.display = '';
    });
    AllWordPlace(); // 重新排列所有词条
    isRandomMode.value = false;
    buttonText.value = "数量";
  }
};

// 数量/重排功能
const CouWord = () => {
  if (isRandomMode.value) {
    // 重排功能：重新排列所有词条（包括隐藏的）
    AllWordPlace();
  } else {
    // 设置随机显示数量
    const val = prompt('请输入每次随机显示的词条数量', randomCount.value);
    const num = parseInt(val);
    if (!isNaN(num) && num > 0) {
      randomCount.value = num;
    }
  }
};
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 60px;
    background-color: rgb(236, 230, 230);
    display: flex;
    gap: 8px;
    align-items: center;
    padding-bottom: 10px;
}

header>div {
    width: 50px;
    height: 50px;
    border-radius: 100%;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 0 2px 5px rgb(151, 151, 151);
}

header>div:hover,
main>.word:hover {
    transform: scale(1.2);
    transition: transform 0.3s ease;
}

.InfBox {
    background-color: red;
}

.AddBox {
    background-color: orange;
}

.RanBox {
    background-color: rgb(234, 234, 31);
}

.CouBox {
    background-color: green;
}

.DelBox {
    background-color: aqua;
}

.CleBox {
    background-color: blue;
}

main {
    height: 100vh;
    background-color: rgb(255, 255, 255);
    position: relative;
    margin-top: 50px;
}

.add,
.mod {
    position: absolute;
    background-color: rgb(255, 255, 255);
    border-radius: 15px;
    height: auto;
    width: auto;
    z-index: 1000;
    border: solid gray;
    box-shadow: 2px 2px 5px gray;
    padding: 30px;
    margin: auto;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
}

.TextInput {
    width: 100%;
    border: 1px solid #ddd;
    margin-bottom: 15px;
    border-radius: 5px;
    padding: 10px;
    font-size: 16px;
}

.AreaInput {
    width: 100%;
    border: 1px solid #ddd;
    margin-bottom: 15px;
    border-radius: 5px;
    padding: 10px;
    font-size: 16px;
    height: 150px;
    resize: vertical;
}

.SubBtn {
    background-color: rgb(0, 242, 255);
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
}

.CloBtn {
    width: 30px;
    height: 30px;
    position: absolute;
    right: -11px;
    top: -11px;
    background-color: #e13e22;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 15px;
    cursor: pointer;
}

.word {
    position: absolute;
    width: auto;
    height: auto;
    padding: 15px;
    background-color: #45adf2;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    font-weight: bold;
    border-radius: 15px;
    cursor: pointer;
    transition: transform 0.3s ease;
    box-shadow: 2px 2px 5px gray;
}
</style>
