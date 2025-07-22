<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      background-color: #f0f0f0;
      color: #333;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    .container {
      max-width: 800px;
      margin: auto;
      background-color: white;
      border-radius: 12px;
      box-shadow: 0 0 10px #ccc;
      padding: 20px;
    }
    h2 {
      color: #222;
    }
    label, select, input {
      display: block;
      margin-bottom: 10px;
      font-size: 16px;
    }
    select, input {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }
    button {
      background-color: #333;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin: 10px 0;
    }
    button:hover {
      background-color: #555;
    }
    .result {
      background-color: #fafafa;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 15px;
      margin-top: 20px;
    }
  </style>
</head>
<body>
<div class="container">
  <h2>حاسبة الأسعار</h2>

  <label>الفئة:</label>
  <select id="category" onchange="updateTypes()">
    <option value="">-- اختر --</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب السحب">أبواب السحب</option>
  </select>

  <label>النوع:</label>
  <select id="type"></select>

  <label>النوع الفرعي (إن وجد):</label>
  <select id="subtype"></select>

  <label>الطول (متر):</label>
  <input type="number" id="length" step="0.01" />

  <label>العرض (متر):</label>
  <input type="number" id="width" step="0.01" />

  <label>الكمية:</label>
  <input type="number" id="quantity" value="1" min="1" />

  <label>هل ترغب بإضافة ستارة؟</label>
  <select id="curtain" onchange="toggleCurtainInputs()">
    <option value="no">بدون</option>
    <option value="yes">نعم</option>
  </select>

  <div id="curtainInputs" style="display:none;">
    <label>طول الستارة (م):</label>
    <input type="number" id="curtainLength" step="0.01" />
    <label>عرض الستارة (م):</label>
    <input type="number" id="curtainWidth" step="0.01" />
  </div>

  <button onclick="calculate()">احسب</button>
  <button onclick="clearResults()">🗑️ محو النتائج</button>
  <button onclick="saveAsWord()">💾 حفظ النتائج كـ Word</button>

  <div id="results" class="result"></div>
</div>

<script>
const prices = {
  "نوافذ": {
    "دبل جلاس دبل فريم": { "ثابت": 34, "حركة": 73, "حركتين": 92, "cbm": 0.13 },
    "دبل جلاس سنجل فريم": { "ثابت": 26, "حركة": 46, "حركتين": 58, "cbm": 0.13 },
    "سنجل جلاس سنجل فريم": { "ثابت": 20, "حركة": 43, "حركتين": 47, "cbm": 0.13 },
    "نوافذ السلايدنج": { "سعر": 10, "cbm": 0.13 },
    "النوافذ الكهربائية": { "سعر": 102, "cbm": 0.13 },
    "سكاي لايت": { "بدون الماكينة": 56, "مع الماكينة": 145, "cbm": 0.13 },
    "كارتن وول": { "ثقيل": 56, "خفيف": 45, "cbm": 0.13 }
  },
  "أبواب": {
    "WPC": { "فارغ": 45, "مع خشب": 50, "مع حشوة ضد الصوت": 60, "مع فريم ألمنيوم": 67, "cbm": 0.11 },
    "أبواب ألمنيوم": { "فارغ": 65, "مع خشب": 75, "فل ألمنيوم": 85, "مخفي": 110, "خارجي": 61, "cbm": 0.11 },
    "أبواب دورات المياه": { "جديد": 55, "قديم": 45, "مخفي زجاجي": 65, "cbm": 0.11 },
    "أبواب المداخل": { "زينك": 66, "ستينلس ستيل": 120, "كاست ألمنيوم": 168, "cbm": 0.2 },
    "أبواب الحدائق": { "كاست ألمنيوم": 91, "cbm": 0.2 },
    "حواجز": { "درج": 43, "حمام": 32, "cbm": 0.05 }
  },
  "أبواب السحب": {
    "داخلي زجاج": 38,
    "داخلي متين": 41,
    "خارجي جزء مفتوح": 55,
    "خارجي جزئين مفتوح": 58,
    "WPC سلايد": 61,
    "cbm": 0.13,
    "cbm_wpc": 0.11
  }
};

function updateTypes() {
  const category = document.getElementById("category").value;
  const typeSelect = document.getElementById("type");
  typeSelect.innerHTML = "<option value=''>-- اختر النوع --</option>";
  const types = Object.keys(prices[category] || {});
  types.forEach(type => {
    typeSelect.innerHTML += `<option value="${type}">${type}</option>`;
  });
  document.getElementById("subtype").innerHTML = "";
}

function toggleCurtainInputs() {
  const show = document.getElementById("curtain").value === "yes";
  document.getElementById("curtainInputs").style.display = show ? "block" : "none";
}

let allResults = [];

function calculate() {
  const cat = document.getElementById("category").value;
  const type = document.getElementById("type").value;
  const subtype = document.getElementById("subtype").value;
  const l = parseFloat(document.getElementById("length").value);
  const w = parseFloat(document.getElementById("width").value);
  const q = parseInt(document.getElementById("quantity").value);
  const curtain = document.getElementById("curtain").value === "yes";
  const curtainL = parseFloat(document.getElementById("curtainLength").value || 0);
  const curtainW = parseFloat(document.getElementById("curtainWidth").value || 0);

  let area = l * w;
  let price = 0;
  let cbm = 0;

  if (cat === "نوافذ") {
    if (type === "نوافذ السلايدنج") {
      price = area * 10 + prices[cat][type].سعر;
      cbm = prices[cat][type].cbm;
    } else if (type === "سكاي لايت") {
      price = area * prices[cat][type][subtype];
      cbm = prices[cat][type].cbm;
    } else if (type === "كارتن وول") {
      price = area * prices[cat][type][subtype];
      cbm = prices[cat][type].cbm;
    } else {
      price = area * prices[cat][type][subtype];
      cbm = prices[cat][type].cbm;
    }
  } else if (cat === "أبواب") {
    price = prices[cat][type][subtype];
    cbm = prices[cat][type].cbm;
  } else if (cat === "أبواب السحب") {
    price = area * prices[cat][type];
    cbm = type.includes("WPC") ? prices[cat].cbm_wpc : prices[cat].cbm;
  }

  let shipping = area * cbm * 48;
  let total = (price + shipping) * q;

  if (curtain) {
    let curtainArea = curtainL * curtainW;
    total += curtainArea * 26;
  }

  let result = `
    <strong>النوع:</strong> ${type} ${subtype || ""}<br>
    <strong>المقاس:</strong> ${l} × ${w} متر<br>
    <strong>الكمية:</strong> ${q}<br>
    <strong>الشحن:</strong> ${shipping.toFixed(2)} ريال<br>
    <strong>السعر الكلي:</strong> ${total.toFixed(2)} ريال<br><hr>
  `;
  allResults.push(result);
  document.getElementById("results").innerHTML = allResults.join("<br>");
}

function clearResults() {
  allResults = [];
  document.getElementById("results").innerHTML = "";
}

function saveAsWord() {
  const header = "<html xmlns:o='urn:schemas-microsoft-com:office:office' " +
                 "xmlns:w='urn:schemas-microsoft-com:office:word' " +
                 "xmlns='http://www.w3.org/TR/REC-html40'>" +
                 "<head><meta charset='utf-8'></head><body>";
  const content = document.getElementById("results").innerHTML;
  const footer = "</body></html>";
  const sourceHTML = header + content + footer;

  const blob = new Blob(['\ufeff', sourceHTML], {
    type: 'application/msword'
  });

  const url = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
  const link = document.createElement("a");
  link.href = url;
  link.download = "النتائج.doc";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>
</body>
</html>
