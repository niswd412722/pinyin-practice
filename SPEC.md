# 拼音打字练习

随机显示词语，输入对应拼音，即时反馈对错。

```json
{
  "name": "拼音打字练习",
  "version": "1.0.0",
  "description": "练习拼音输入的打字工具",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

```javascript
// server.js
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'public')));

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(3000, () => {
  console.log('拼音练习已启动: http://localhost:3000');
});
```

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>拼音打字练习</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background: #1a1a2e;
      color: #eee;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 40px 20px;
    }
    h1 { font-size: 24px; margin-bottom: 8px; color: #e94560; }
    .subtitle { color: #888; margin-bottom: 40px; font-size: 14px; }

    .stats {
      display: flex;
      gap: 40px;
      margin-bottom: 30px;
    }
    .stat { text-align: center; }
    .stat-value { font-size: 32px; font-weight: bold; color: #e94560; }
    .stat-label { font-size: 12px; color: #888; margin-top: 4px; }

    .word-card {
      background: #16213e;
      border: 2px solid #e94560;
      border-radius: 16px;
      padding: 40px 60px;
      text-align: center;
      margin-bottom: 24px;
    }
    .word {
      font-size: 56px;
      font-weight: bold;
      letter-spacing: 8px;
      color: #fff;
    }
    .hint {
      margin-top: 12px;
      font-size: 14px;
      color: #666;
      height: 20px;
    }

    .input-area {
      width: 100%;
      max-width: 400px;
    }
    #pinyin-input {
      width: 100%;
      padding: 16px 24px;
      font-size: 24px;
      text-align: center;
      border: 2px solid #333;
      border-radius: 12px;
      background: #16213e;
      color: #fff;
      outline: none;
      transition: border-color 0.2s;
    }
    #pinyin-input:focus { border-color: #e94560; }
    #pinyin-input.correct { border-color: #4ade80; background: rgba(74, 222, 128, 0.1); }
    #pinyin-input.wrong { border-color: #f87171; background: rgba(248, 113, 113, 0.1); }

    .feedback {
      height: 28px;
      margin-top: 16px;
      font-size: 16px;
      text-align: center;
    }
    .feedback.ok { color: #4ade80; }
    .feedback.fail { color: #f87171; }

    .controls {
      margin-top: 40px;
      display: flex;
      gap: 16px;
      flex-wrap: wrap;
      justify-content: center;
    }
    button {
      padding: 10px 24px;
      border: none;
      border-radius: 8px;
      font-size: 14px;
      cursor: pointer;
      background: #e94560;
      color: #fff;
      transition: opacity 0.2s;
    }
    button:hover { opacity: 0.85; }
    button.secondary { background: #333; }

    .level-select {
      display: flex;
      gap: 8px;
      margin-bottom: 24px;
      flex-wrap: wrap;
      justify-content: center;
    }
    .level-btn {
      padding: 6px 16px;
      border: 1px solid #444;
      background: transparent;
      color: #888;
      font-size: 13px;
      border-radius: 20px;
      cursor: pointer;
    }
    .level-btn.active {
      border-color: #e94560;
      color: #e94560;
      background: rgba(233, 69, 96, 0.1);
    }
  </style>
</head>
<body>
  <h1>拼音打字练习</h1>
  <p class="subtitle">输入正确拼音，空格或回车下一题</p>

  <div class="level-select">
    <button class="level-btn active" data-level="1">简单</button>
    <button class="level-btn" data-level="2">常用</button>
    <button class="level-btn" data-level="3">成语</button>
  </div>

  <div class="stats">
    <div class="stat"><div class="stat-value" id="correct">0</div><div class="stat-label">正确</div></div>
    <div class="stat"><div class="stat-value" id="total">0</div><div class="stat-label">总数</div></div>
    <div class="stat"><div class="stat-value" id="rate">0%</div><div class="stat-label">正确率</div></div>
  </div>

  <div class="word-card">
    <div class="word" id="word">词语</div>
    <div class="hint" id="hint"></div>
  </div>

  <div class="input-area">
    <input type="text" id="pinyin-input" placeholder="输入拼音" autocomplete="off" autofocus>
    <div class="feedback" id="feedback"></div>
  </div>

  <div class="controls">
    <button onclick="nextWord()">下一题 (空格)</button>
    <button class="secondary" onclick="resetStats()">重置</button>
  </div>

  <script>
    const WORDS = {
      1: [
        { w: '妈妈', p: 'mā ma' }, { w: '爸爸', p: 'bà ba' }, { w: '学习', p: 'xué xí' },
        { w: '中国', p: 'zhōng guó' }, { w: '朋友', p: 'péng you' }, { w: '工作', p: 'gōng zuò' },
        { w: '时间', p: 'shí jiān' }, { w: '快乐', p: 'kuài lè' }, { w: '大家', p: 'dà jiā' },
        { w: '什么', p: 'shén me' }, { w: '知道', p: 'zhī dào' }, { w: '可以', p: 'kě yǐ' },
        { w: '喜欢', p: 'xǐ huān' }, { w: '问题', p: 'wèn tí' }, { w: '谢谢', p: 'xiè xiè' },
        { w: '老师', p: 'lǎo shī' }, { w: '学生', p: 'xué shēng' }, { w: '电脑', p: 'diàn nǎo' },
        { w: '手机', p: 'shǒu jī' }, { w: '天气', p: 'tiān qì' }, { w: '吃饭', p: 'chī fàn' },
        { w: '睡觉', p: 'shuì jiào' }, { w: '看书', p: 'kàn shū' }, { w: '写字', p: 'xiě zì' },
        { w: '说话', p: 'shuō huà' }, { w: '走路', p: 'zǒu lù' }, { w: '跑步', p: 'pǎo bù' },
        { w: '喝水', p: 'hē shuǐ' }, { w: '开门', p: 'kāi mén' }, { w: '关门', p: 'guān mén' },
      ],
      2: [
        { w: '北京', p: 'běi jīng' }, { w: '上海', p: 'shàng hǎi' }, { w: '广州', p: 'guǎng zhōu' },
        { w: '医院', p: 'yī yuàn' }, { w: '学校', p: 'xué xiào' }, { w: '银行', p: 'yín háng' },
        { w: '火车', p: 'huǒ chē' }, { w: '飞机', p: 'fēi jī' }, { w: '天气', p: 'tiān qì' },
        { w: '水果', p: 'shuǐ guǒ' }, { w: '蔬菜', p: 'shū cài' }, { w: '米饭', p: 'mǐ fàn' },
        { w: '面条', p: 'miàn tiáo' }, { w: '衣服', p: 'yī fu' }, { w: '裤子', p: 'kù zi' },
        { w: '鞋子', p: 'xié zi' }, { w: '帽子', p: 'mào zi' }, { w: '眼睛', p: 'yǎn jing' },
        { w: '耳朵', p: 'ěr duo' }, { w: '鼻子', p: 'bí zi' }, { w: '嘴巴', p: 'zuǐ ba' },
        { w: '手', p: 'shǒu' }, { w: '脚', p: 'jiǎo' }, { w: '头', p: 'tóu' },
        { w: '今天', p: 'jīn tiān' }, { w: '明天', p: 'míng tiān' }, { w: '昨天', p: 'zuó tiān' },
        { w: '早上', p: 'zǎo shang' }, { w: '晚上', p: 'wǎn shang' }, { w: '中午', p: 'zhōng wǔ' },
      ],
      3: [
        { w: '一心一意', p: 'yī xīn yī yì' }, { w: '三心二意', p: 'sān xīn èr yì' },
        { w: '五颜六色', p: 'wǔ yán liù sè' }, { w: '七上八下', p: 'qī shàng bā xià' },
        { w: '十全十美', p: 'shí quán shí měi' }, { w: '百花齐放', p: 'bǎi huā qí fàng' },
        { w: '千言万语', p: 'qiān yán wàn yǔ' }, { w: '惊天动地', p: 'jīng tiān dòng dì' },
        { w: '东张西望', p: 'dōng zhāng xī wàng' }, { w: '南来北往', p: 'nán lái běi wǎng' },
        { w: '前因后果', p: 'qián yīn hòu guǒ' }, { w: '左邻右舍', p: 'zuǒ lín yòu shè' },
        { w: '上行下效', p: 'shàng xíng xià xiào' }, { w: '大同小异', p: 'dà tóng xiǎo yì' },
        { w: '大材小用', p: 'dà cái xiǎo yòng' }, { w: '眼高手低', p: 'yǎn gāo shǒu dī' },
        { w: '口是心非', p: 'kǒu shì xīn fēi' }, { w: '有气无力', p: 'yǒu qì wú lì' },
        { w: '没完没了', p: 'méi wán méi liǎo' }, { w: '说三道四', p: 'shuō sān dào sì' },
        { w: '能说会道', p: 'néng shuō huì dào' }, { w: '取长补短', p: 'qǔ cháng bǔ duǎn' },
        { w: '丢三落四', p: 'diū sān là sì' }, { w: '举一反三', p: 'jǔ yī fǎn sān' },
        { w: '三长两短', p: 'sān cháng liǎng duǎn' }, { w: '不三不四', p: 'bù sān bù sì' },
        { w: '低三下四', p: 'dī sān xià sì' }, { w: '朝三暮四', p: 'zhāo sān mù sì' },
        { w: '四面八方', p: 'sì miàn bā fāng' }, { w: '五湖四海', p: 'wǔ hú sì hǎi' },
      ]
    };

    let level = 1;
    let currentWord = null;
    let correct = 0;
    let total = 0;
    let used = [];

    const wordEl = document.getElementById('word');
    const hintEl = document.getElementById('hint');
    const inputEl = document.getElementById('pinyin-input');
    const feedbackEl = document.getElementById('feedback');
    const correctEl = document.getElementById('correct');
    const totalEl = document.getElementById('total');
    const rateEl = document.getElementById('rate');

    function pickWord() {
      const pool = WORDS[level].filter(w => !used.includes(w.w));
      if (pool.length === 0) used = [];
      const pool2 = pool.length ? pool : WORDS[level];
      const item = pool2[Math.floor(Math.random() * pool2.length)];
      used.push(item.w);
      return item;
    }

    function showWord() {
      currentWord = pickWord();
      wordEl.textContent = currentWord.w;
      hintEl.textContent = '';
      inputEl.value = '';
      inputEl.className = '';
      feedbackEl.textContent = '';
      inputEl.focus();
    }

    function checkAnswer() {
      const input = inputEl.value.trim().toLowerCase();
      const target = currentWord.p.toLowerCase();
      total++;
      if (input === target) {
        correct++;
        inputEl.className = 'correct';
        feedbackEl.textContent = '✓ 正确！';
        feedbackEl.className = 'feedback ok';
        setTimeout(nextWord, 800);
      } else {
        inputEl.className = 'wrong';
        hintEl.textContent = `正确答案: ${currentWord.p}`;
        feedbackEl.textContent = '✗ 再试一次，或按空格下一题';
        feedbackEl.className = 'feedback fail';
      }
      updateStats();
    }

    function updateStats() {
      correctEl.textContent = correct;
      totalEl.textContent = total;
      rateEl.textContent = total > 0 ? Math.round(correct / total * 100) + '%' : '0%';
    }

    function nextWord() {
      showWord();
    }

    function resetStats() {
      correct = 0;
      total = 0;
      used = [];
      updateStats();
      showWord();
    }

    document.querySelectorAll('.level-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        document.querySelectorAll('.level-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        level = parseInt(btn.dataset.level);
        used = [];
        showWord();
      });
    });

    inputEl.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        if (inputEl.classList.contains('wrong')) {
          nextWord();
        } else {
          checkAnswer();
        }
      }
    });

    document.addEventListener('keydown', (e) => {
      if (e.key === ' ' && document.activeElement !== inputEl) {
        e.preventDefault();
        nextWord();
      }
    });

    showWord();
  </script>
</body>
</html>
```
