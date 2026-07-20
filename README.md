# NPQ-Chinese

<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>東區尤德夫人那打素醫院 物理治療部 - Northwick Park 頸痛問卷（NPQ）</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: "Noto Sans TC", "Microsoft JhengHei", sans-serif; background:#f5f5f5; margin:0; padding:16px; }
    .container { max-width:900px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.1); }
    
    .hospital-title { text-align:center; font-size: 1.35rem; color: #555; margin-bottom: 6px; font-weight: 500; }
    .main-title { text-align:center; margin-top: 0; margin-bottom: 20px; font-size: 1.85rem; color: #111; }
    
    fieldset.question { margin-bottom:20px; padding:12px; border:1px solid #ddd; border-radius:8px; background:#fafafa; }
    legend.question-title { font-weight:bold; margin-bottom:8px; padding:0 6px; float:none; width:auto; }
    
    .options { display:flex; flex-direction:column; gap:6px; clear:both; padding-top:4px; }
    .option { display:flex; align-items:center; gap:8px; padding:8px 12px; border-radius:6px; background:#fff; border:1px solid #ccc; cursor:pointer; font-size:0.95rem; transition: background 0.2s, border-color 0.2s; }
    .option input { cursor:pointer; }
    
    .option:hover { background: #f0f7ff; border-color: #1976d2; }
    .option input:focus-visible { outline: 2px solid #1976d2; }
    
    .clear-btn { align-self: flex-start; margin-top: 4px; padding: 6px 12px; font-size: 0.85rem; background: #e0e0e0; color: #333; border: none; border-radius: 4px; cursor: pointer; transition: background 0.2s; }
    .clear-btn:hover { background: #d5d5d5; }

    button.calc-btn { margin-top:20px; padding:12px 20px; border:none; border-radius:6px; background:#1976d2; color:white; font-size:1rem; cursor:pointer; width: 100%; max-width: 200px; display: block; margin-left: auto; margin-right: auto; }
    button.calc-btn:hover { background:#125a9c; }
    
    .modal { display:none; position:fixed; z-index:1000; inset:0; background:rgba(0,0,0,0.4); }
    .modal-content { background:#fff; margin:15% auto; padding:20px; border-radius:10px; width:90%; max-width:440px; text-align:center; box-shadow:0 2px 8px rgba(0,0,0,0.3); }
    .close-btn { margin-top:15px; padding:10px 16px; background:#555; color:white; border:none; border-radius:6px; cursor:pointer; }
    .close-btn:hover { background:#333; }
    .warning { color:#c62828; margin-top:15px; font-size:1rem; font-weight: bold; text-align: center; }
    .hint { font-size:0.85rem; color:#666; margin-top: 2px; margin-bottom: 8px; }
    
    .notice-box { background: #fffde7; border-left: 4px solid #fbc02d; border-right: 4px solid #fbc02d; padding: 16px 12px; margin-top: 15px; text-align: center; font-size: 1.15rem; color: #5d4037; border-radius: 4px; line-height: 1.6; font-weight: bold; }
  </style>
</head>
<body>

<div class="container">
  <div class="hospital-title" id="inner-h-title">東區尤德夫人那打素醫院 物理治療部</div>
  <h1 class="main-title" id="inner-m-title">Northwick Park 頸痛問卷（NPQ）</h1>
  
  <p>請根據你今日的情況，在每一題中選擇最貼切的描述。<strong>（若平時不駕駛，第9題可留空）</strong></p>
  <div id="questions"></div>
  <button type="button" class="calc-btn" onclick="calculateNPQ()">計算分數</button>
  <div id="warning" class="warning" aria-live="assertive"></div>
</div>

<div id="scoreModal" class="modal" role="dialog" aria-labelledby="modalTitle" aria-hidden="true">
  <div class="modal-content">
    <h2 id="modalTitle">評估結果</h2>
    <p id="rawScoreText"></p>
    <p id="percentScoreText"></p>
    <p style="font-size:0.9rem; color:#555;">頸痛失能指數越低越好<br>（0% 代表無症狀，100% 代表最嚴重失能）</p>

    <div class="notice-box">
      請不要離開此畫面，並出示給您的治療師。<br>
      您亦可以進行螢幕截圖。<br>
      謝謝。
    </div>
    
    <button type="button" class="close-btn" onclick="closeModal()">關閉</button>
  </div>
</div>

<script>
window.addEventListener('DOMContentLoaded', () => {
  const headers = document.querySelectorAll('h1, h2, div');
  headers.forEach(el => {
    if (el.textContent.trim() === 'NPQ-Chinese') {
      el.innerHTML = '<span style="font-size: 1.35rem; color: #555; font-weight: 500; display:block; text-align:center;">東區尤德夫人那打素醫院 物理治療部</span>';
      const innerHTitle = document.getElementById('inner-h-title');
      if (innerHTitle) innerHTitle.style.display = 'none';
    }
  });
});

const questions = [
  { title: "1. 現在頸痛的程度", options: ["沒有頸痛", "溫和", "中等", "很厲害", "簡直不可想像"] },
  { title: "2. 頸痛與睡眠", options: ["頸痛從不干擾我睡眠", "頸痛有時會干擾我睡眠", "頸痛經常干擾我睡眠", "頸痛使我每晚睡眠少於五小時", "頸痛使我每晚睡眠少於二小時"] },
  { title: "3. 手臂在夜晚感到發麻或針刺", options: ["夜晚沒有麻痺或針刺感", "偶爾有麻痺或針刺感", "經常干擾睡眠", "使我每晚睡眠少於五小時", "使我每晚睡眠少於二小時"] },
  { title: "4. 每天症狀持續的時間", options: ["整天都正常", "起床後不適少於一小時", "不適持續 1–4 小時", "不適持續超過 4 小時", "整天持續不適"] },
  { title: "5. 攜帶物件", options: ["可攜帶重物而不痛", "可攜帶重物但會痛", "只能攜帶中等重量物件", "只能拿起輕物件", "甚麼都拿不起來"] },
  { title: "6. 閱讀及看電視", options: ["多久都可以，沒有困難", "姿勢適當時可持續", "可持續但會痛", "痛使我提早結束活動", "痛使我無法進行"] },
  { title: "7. 工作、家務之類", options: ["可做平時工作而不痛", "可做但會痛", "只能做一半工作量", "只能做四分之一工作量", "無法工作"] },
  { title: "8. 社交活動", options: ["社交正常，無痛", "社交正常但會痛", "受限但仍可外出", "只能在家社交", "無社交生活"] },
  { title: "9. 駕駛（可略過）", options: ["可駕駛，無不適", "可駕駛但有不適", "偶爾受限", "經常受限", "無法駕駛"], optional: true }
];

const questionsDiv = document.getElementById("questions");

questions.forEach((q, index) => {
  const qNum = index + 1;
  const fieldset = document.createElement("fieldset");
  fieldset.className = "question";
  if (q.optional) {
    fieldset.id = `optional-fieldset-${qNum}`;
  }

  const legend = document.createElement("legend");
  legend.className = "question-title";
  legend.textContent = q.title;
  fieldset.appendChild(legend);

  if (q.optional) {
    const hint = document.createElement("div");
    hint.className = "hint";
    hint.textContent = "（若不駕駛，此題可不填）";
    fieldset.appendChild(hint);
  }

  const optionsDiv = document.createElement("div");
  optionsDiv.className = "options";

  q.options.forEach((opt, i) => {
    const label = document.createElement("label");
    label.className = "option";

    const input = document.createElement("input");
    input.type = "radio";
    input.name = `q${qNum}`;
    input.value = i;

    label.appendChild(input);
    label.appendChild(document.createTextNode(opt));
    optionsDiv.appendChild(label);
  });

  if (q.optional) {
    const clearButton = document.createElement("button");
    clearButton.type = "button";
    clearButton.className = "clear-btn";
    clearButton.textContent = "清除此題（我不駕駛）";
    clearButton.setAttribute("aria-controls", `optional-fieldset-${qNum}`);
    clearButton.onclick = function() {
      const radios = document.querySelectorAll(`input[name="q${qNum}"]`);
      radios.forEach(radio => radio.checked = false);
    };
    optionsDiv.appendChild(clearButton);
  }

  fieldset.appendChild(optionsDiv);
  questionsDiv.appendChild(fieldset);
});

function calculateNPQ() {
  const warning = document.getElementById("warning");
  warning.textContent = "";

  let totalScore = 0;
  let maxPossibleScore = 0;

  for (let i = 1; i <= 9; i++) {
    const selected = document.querySelector(`input[name="q${i}"]:checked`);
    const isOptional = questions[i-1].optional;
    
    if (!selected) {
      if (isOptional) {
        continue; 
      } else {
        warning.textContent = `請填寫第 ${i} 題後再進行計算。`;
        document.querySelector(`input[name="q${i}"]`).focus();
        return;
      }
    }
    
    totalScore += parseInt(selected.value);
    maxPossibleScore += 4; 
  }

  const percentage = Math.round((totalScore / maxPossibleScore) * 100);

  document.getElementById("rawScoreText").innerHTML = `原始總分：<strong>${totalScore}</strong> / ${maxPossibleScore}`;
  document.getElementById("percentScoreText").innerHTML = `頸痛失能指數：<strong>${percentage}%</strong>`;

  const modal = document.getElementById("scoreModal");
  modal.style.display = "block";
  modal.setAttribute("aria-hidden", "false");
}

function closeModal() {
  const modal = document.getElementById("scoreModal");
  modal.style.display = "none";
  modal.setAttribute("aria-hidden", "true");
}
</script>

</body>
</html>
