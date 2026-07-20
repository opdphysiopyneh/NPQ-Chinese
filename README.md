# NPQ-Chinese
NPQ Chinese Questionnaire

<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>Northwick Park 頸痛問卷（NPQ）</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: "Noto Sans TC", "Microsoft JhengHei", sans-serif; background:#f5f5f5; margin:0; padding:16px; }
    .container { max-width:900px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.1); }
    h1 { text-align:center; margin-bottom:20px; }
    .question { margin-bottom:20px; padding:12px; border:1px solid #ddd; border-radius:8px; background:#fafafa; }
    .question-title { font-weight:bold; margin-bottom:8px; }
    .options { display:flex; flex-direction:column; gap:6px; }
    .option { display:flex; align-items:center; gap:8px; padding:6px 10px; border-radius:6px; background:#fff; border:1px solid #ccc; cursor:pointer; font-size:0.9rem; }
    .option input { cursor:pointer; }
    button { margin-top:20px; padding:12px 20px; border:none; border-radius:6px; background:#1976d2; color:white; font-size:1rem; cursor:pointer; }
    button:hover { background:#125a9c; }
    .modal { display:none; position:fixed; z-index:1000; inset:0; background:rgba(0,0,0,0.4); }
    .modal-content { background:#fff; margin:15% auto; padding:20px; border-radius:10px; width:90%; max-width:420px; text-align:center; box-shadow:0 2px 8px rgba(0,0,0,0.3); }
    .close-btn { margin-top:15px; padding:10px 16px; background:#555; color:white; border:none; border-radius:6px; cursor:pointer; }
    .close-btn:hover { background:#333; }
    .warning { color:#c62828; margin-top:10px; font-size:0.9rem; }
  </style>
</head>
<body>

<div class="container">
  <h1>Northwick Park 頸痛問卷（NPQ）</h1>
  <p>請根據你今日的情況，在每一題中選擇最貼切的描述。</p>
  <div id="questions"></div>
  <button onclick="calculateNPQ()">計算分數</button>
  <div id="warning" class="warning"></div>
</div>

<div id="scoreModal" class="modal">
  <div class="modal-content">
    <h2>評估結果</h2>
    <p id="rawScoreText"></p>
    <p id="percentScoreText"></p>
    <p style="font-size:0.9rem; color:#555;">轉換分數越低表示功能越好（最高 100 分）</p>
    <button class="close-btn" onclick="closeModal()">關閉</button>
  </div>
</div>

<script>
const questions = [
  {
    title: "1. 現在頸痛的程度",
    options: ["沒有頸痛", "溫和", "中等", "很厲害", "簡直不可想像"]
  },
  {
    title: "2. 頸痛與睡眠",
    options: ["頸痛從不干擾我睡眠", "頸痛有時會干擾我睡眠", "頸痛經常干擾我睡眠", "頸痛使我每晚睡眠少於五小時", "頸痛使我每晚睡眠少於兩小時"]
  },
  {
    title: "3. 手臂在夜晚感到發麻或針刺",
    options: ["夜晚沒有麻痺或針刺感", "偶爾有麻痺或針刺感", "經常干擾睡眠", "使我每晚睡眠少於五小時", "使我每晚睡眠少於兩小時"]
  },
  {
    title: "4. 每天症狀持續的時間",
    options: ["整天都正常", "起床後不適少於一小時", "不適持續 1–4 小時", "不適持續超過 4 小時", "整天持續不適"]
  },
  {
    title: "5. 攜帶物件",
    options: ["可攜帶重物而不痛", "可攜帶重物但會痛", "只能攜帶中等重量物件", "只能拿起輕物件", "甚麼都拿不起來"]
  },
  {
    title: "6. 閱讀及看電視",
    options: ["多久都可以，沒有困難", "姿勢適當時可持續", "可持續但會痛", "痛使我提早結束活動", "痛使我無法進行"]
  },
  {
    title: "7. 工作、家務之類",
    options: ["可做平時工作而不痛", "可做但會痛", "只能做一半工作量", "只能做四分之一工作量", "無法工作"]
  },
  {
    title: "8. 社交活動",
    options: ["社交正常，無痛", "社交正常但會痛", "受限但仍可外出", "只能在家社交", "無社交生活"]
  },
  {
    title: "9. 駕駛（如健康時也不駕駛可略過）",
    options: ["可駕駛，無不適", "可駕駛但有不適", "偶爾受限", "經常受限", "無法駕駛"]
  }
];

const questionsDiv = document.getElementById("questions");

questions.forEach((q, index) => {
  const qNum = index + 1;
  const qDiv = document.createElement("div");
  qDiv.className = "question";

  const title = document.createElement("div");
  title.className = "question-title";
  title.textContent = q.title;
  qDiv.appendChild(title);

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

  qDiv.appendChild(optionsDiv);
  questionsDiv.appendChild(qDiv);
});

function calculateNPQ() {
  const warning = document.getElementById("warning");
  warning.textContent = "";

  let total = 0;
  let count = 0;

  for (let i = 1; i <= 9; i++) {
    const selected = document.querySelector(`input[name="q${i}"]:checked`);
    if (!selected) {
      warning.textContent = "請完成所有題目後再計算分數。";
      return;
    }
    total += parseInt(selected.value);
    count++;
  }

  const maxScore = 36;
  const percent = Math.round(100 - (total / maxScore * 100));

  document.getElementById("rawScoreText").innerHTML = `原始總分：<strong>${total}</strong> / ${maxScore}`;
  document.getElementById("percentScoreText").innerHTML = `轉換分數：<strong>${percent}</strong> 分`;

  document.getElementById("scoreModal").style.display = "block";
}

function closeModal() {
  document.getElementById("scoreModal").style.display = "none";
}
</script>

</body>
</html>
