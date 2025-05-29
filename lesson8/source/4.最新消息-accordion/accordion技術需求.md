# Accordion 元件技術需求規格

本文件詳細說明了 Accordion (手風琴) 元件的 HTML、CSS 和 JavaScript 技術需求。

## 1. 功能概述

- **基本功能**：使用者可以點擊標題區域來展開或摺疊對應的內容區塊。
- **標題與日期**：每個 Accordion 項目都會同時顯示標題和日期。
- **預設狀態**：預設情況下，第一個 Accordion 項目的內容會是展開狀態。
- **多行標題**：標題內容可能較長，需要支援多行顯示。
- **固定高度內容**：內容區域將具有固定的高度（或限制顯示行數），超出部分將被隱藏或提供滾動條。
- **資料來源**：所有 Accordion 的資料（標題、日期、內容）將從資料庫動態獲取。
- **響應式設計 (RWD)**：元件需在桌面和行動裝置上均有良好顯示效果。
- **Jinja 整合**：設計需考慮未來手動整合至 Jinja 模板的便利性。

## 2. HTML 結構

以下是建議的 HTML 結構。此結構易於使用 Jinja 等模板引擎進行動態渲染。

```html
<div class="accordion-container">
  <!-- 單個 Accordion 項目 (可由後端迴圈產生) -->
  <div class="accordion-item">
    <button id="accordion-header-1" class="accordion-header" aria-expanded="false" aria-controls="accordion-content-1">
      <span class="accordion-title">第一個標題內容可能會非常長長長長長長長長長長長長長長長長長長長長</span>
      <span class="accordion-date">2023-10-26</span>
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
    <div id="accordion-content-1" class="accordion-content" role="region" aria-labelledby="accordion-header-1">
      <p>這是第一個項目的詳細內容。這部分內容將會有固定的高度限制，例如顯示固定的行數。</p>
      <p>更多內容...</p>
    </div>
  </div>

  <div class="accordion-item">
    <button id="accordion-header-2" class="accordion-header" aria-expanded="false" aria-controls="accordion-content-2">
      <span class="accordion-title">第二個標題</span>
      <span class="accordion-date">2023-10-27</span>
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
    <div id="accordion-content-2" class="accordion-content" role="region" aria-labelledby="accordion-header-2">
      <p>這是第二個項目的詳細內容。</p>
    </div>
  </div>

  <!-- 更多 Accordion 項目 -->
</div>
```

**結構說明：**
- `.accordion-container`: Accordion 的主容器。
- `.accordion-item`: 代表每一個可摺疊的項目。
- `.accordion-header`: 項目的標題區域，也是觸發展開/摺疊的按鈕。
  - 使用 `<button>` 元素以確保可訪問性和鍵盤操作。
  - `id`: 唯一標識符，被內容面板的 `aria-labelledby` 引用。
  - `aria-expanded`: 指示內容是否展開。
  - `aria-controls`: 指向對應的內容面板 ID。
- `.accordion-title`: 顯示標題文字。
- `.accordion-date`: 顯示日期。
- `.accordion-icon`: 顯示展開/摺疊狀態的圖示 (例如 `+` / `-`)。
- `.accordion-content`: 包含詳細內容的面板。
  - `id`: 唯一標識符，被 `aria-controls` 引用。
  - `role="region"` 和 `aria-labelledby`: 增強可訪問性。

**Jinja 整合考量：**
在 Jinja 模板中，您可以使用 `for` 迴圈遍歷資料列表來生成 `.accordion-item`。
```jinja
<div class="accordion-container">
  {% for item in items %}
  <div class="accordion-item">
    <button id="accordion-header-{{ loop.index }}" class="accordion-header" aria-expanded="{{ 'true' if loop.first else 'false' }}" aria-controls="accordion-content-{{ loop.index }}">
      <span class="accordion-title">{{ item.title }}</span>
      <span class="accordion-date">{{ item.date }}</span>
      <span class="accordion-icon" aria-hidden="true">{% if loop.first %}-{% else %}+{% endif %}</span>
    </button>
    <div id="accordion-content-{{ loop.index }}" class="accordion-content {% if loop.first %}active{% endif %}" role="region" aria-labelledby="accordion-header-{{ loop.index }}">
      <p>{{ item.content }}</p>
    </div>
  </div>
  {% endfor %}
</div>
```
注意：上述 Jinja 範例中，`aria-expanded`、圖示的初始狀態以及內容面板的 `active` class 根據 `loop.first` 設定。JavaScript 仍需處理點擊後的動態變化並確保與初始狀態一致。

## 3. CSS 樣式

以下是基本的 CSS 樣式建議，您可以根據實際設計稿進行調整。

```css
.accordion-container {
  width: 100%;
  max-width: 800px; /* 可依需求調整 */
  margin: 20px auto;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.accordion-item {
  border-bottom: 1px solid #ddd;
}

.accordion-item:last-child {
  border-bottom: none;
}

.accordion-header {
  background-color: #f7f7f7;
  color: #333;
  cursor: pointer;
  padding: 15px 20px;
  width: 100%;
  text-align: left;
  border: none;
  outline: none;
  transition: background-color 0.3s ease;
  display: flex; /* 使用 flexbox 排列標題、日期和圖示 */
  justify-content: space-between; /* 元素間隔開 */
  align-items: center; /* 垂直居中 */
  font-size: 1em; /* 基本字體大小 */
}

.accordion-header:hover,
.accordion-header.active {
  background-color: #e9e9e9;
}

.accordion-title {
  flex-grow: 1; /* 標題佔據多餘空間，允許換行 */
  margin-right: 10px; /* 與日期和圖示間隔 */
  line-height: 1.4; /* 調整行高以適應多行 */
}

.accordion-date {
  font-size: 0.9em;
  color: #666;
  margin-right: 10px; /* 與圖示間隔 */
  white-space: nowrap; /* 防止日期換行 */
  flex-shrink: 0; /* 防止日期在高密度內容下被壓縮 */
}

.accordion-icon {
  font-size: 1.2em;
  font-weight: bold;
  min-width: 20px; /* 給圖示一個最小寬度 */
  text-align: center;
  transition: transform 0.3s ease;
}

.accordion-header.active .accordion-icon {
  /* transform: rotate(45deg); /* 可選：若圖示為 +，旋轉成 x */
}

.accordion-content {
  padding: 0 20px;
  background-color: white;
  max-height: 0; /* 預設摺疊 */
  overflow: hidden;
  transition: max-height 0.3s ease-out, padding 0.3s ease-out;
  /* 固定高度內容 (例如固定顯示 5 行文字) */
  /* line-height: 1.5em; */ /* 設定行高以計算總高度 */
}

.accordion-content.active {
  /* 根據「固定顯示行數」的需求來設定 max-height */
  /* 例如：若行高 1.5em，顯示 5 行，則 max-height 約為 (1.5em * 5) + padding上 + padding下 */
  max-height: 200px; /* 展開時的最大高度，根據內容調整 */
  overflow-y: auto;  /* 如果內容超出此高度，則顯示滾動條 */
  /* 如果不希望滾動，則設為 overflow: hidden; */
  padding: 15px 20px;
  border-top: 1px solid #eee; /* 內容區與頭部之間的分隔線 */
}

.accordion-content p {
  margin: 0 0 10px 0;
  line-height: 1.6;
}
.accordion-content p:last-child {
  margin-bottom: 0;
}

/* 響應式設計 (RWD) */
@media (max-width: 600px) {
  .accordion-header {
    padding: 12px 15px;
    /* 在小螢幕上，可以考慮讓標題和日期垂直排列或換行 */
    /* flex-wrap: wrap; */
  }
  .accordion-title {
    font-size: 0.95em;
    /* margin-bottom: 5px; /* 如果 flex-wrap: wrap */
  }
  .accordion-date {
    font-size: 0.8em;
    /* margin-top: 5px; margin-right: 0; /* 如果 header 改為垂直排列 */
  }
  .accordion-content.active {
    max-height: 300px; /* 手機版可能需要更高的高度或不同的行數限制 */
  }
}
```

**內容固定高度/行數說明：**
- 使用 `max-height` 和 `overflow-y: auto` (或 `overflow: hidden`) 是常見做法。
- `max-height` 的值需要根據設計的行高和希望顯示的行數來計算。例如，如果行高 `line-height: 1.6em;` 且希望顯示 3 行，則內容部分的 `max-height` 大約是 `1.6em * 3` 加上上下 `padding`。
- 另一種方法是使用 `-webkit-line-clamp`，但這主要適用於 WebKit/Blink 核心的瀏覽器，且需要搭配 `display: -webkit-box;` 和 `-webkit-box-orient: vertical;`。
  ```css
  .line-clamp-example {
    display: -webkit-box; /* 或者 inline-block 等 */
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 3; /* 限制為 3 行 */
    overflow: hidden;
    text-overflow: ellipsis;
    /* 可能還需要設定一個固定的 height 或 max-height */
  }
  ```

## 4. JavaScript 行為

以下 JavaScript 程式碼用於處理 Accordion 的展開和摺疊行為。

```javascript
document.addEventListener('DOMContentLoaded', function () {
  const accordionItems = document.querySelectorAll('.accordion-item');

  // (可選功能) 關閉所有其他 Accordion 項目
  function closeAllOtherItems(currentItemHeader) {
    accordionItems.forEach(item => {
      const header = item.querySelector('.accordion-header');
      if (header !== currentItemHeader && header.classList.contains('active')) {
        const content = item.querySelector('.accordion-content');
        const icon = header.querySelector('.accordion-icon');

        header.classList.remove('active');
        header.setAttribute('aria-expanded', 'false');
        content.classList.remove('active');
        // 如果使用 JS 動態設定 max-height，則需取消註解下一行
        // content.style.maxHeight = null;
        if (icon) icon.textContent = '+';
      }
    });
  }

  accordionItems.forEach((item, index) => {
    const header = item.querySelector('.accordion-header');
    const content = item.querySelector('.accordion-content');
    const icon = header.querySelector('.accordion-icon');

    // 預設展開第一個項目 (與 Jinja 模板中的初始狀態對應)
    if (index === 0) {
      // 確保初始狀態與 HTML/Jinja 渲染一致
      if (!header.classList.contains('active')) { // 如果 JS 先執行或 Jinja 未設定
          header.classList.add('active');
          content.classList.add('active');
      }
      header.setAttribute('aria-expanded', 'true');
      if (icon) icon.textContent = '-';
      // CSS 中的 .active class 已設定 max-height。
      // 如果需要 JS 動態計算高度 (例如內容高度不固定時):
      // content.style.maxHeight = content.scrollHeight + "px";
    } else {
      header.setAttribute('aria-expanded', 'false');
      if (icon) icon.textContent = '+';
      // content.style.maxHeight = null; // 確保摺疊 (如果使用 JS 動態設定 max-height)
    }

    header.addEventListener('click', function () {
      const isActive = this.classList.contains('active');

      // **取消註解下一行以啟用「一次只展開一個項目」的行為**
      // closeAllOtherItems(this);

      // 切換目前點擊的項目
      if (isActive) {
        this.classList.remove('active');
        this.setAttribute('aria-expanded', 'false');
        content.classList.remove('active');
        // content.style.maxHeight = null; // 如果使用 JS 動態設定 max-height
        if (icon) icon.textContent = '+';
      } else {
        this.classList.add('active');
        this.setAttribute('aria-expanded', 'true');
        content.classList.add('active');
        // content.style.maxHeight = content.scrollHeight + "px"; // 如果使用 JS 動態設定 max-height
        if (icon) icon.textContent = '-';
      }
    });
  });
});
```

**JavaScript 說明：**
- 監聽 `DOMContentLoaded` 事件以確保 HTML 完全載入。
- 選取所有的 `.accordion-item`。
- 為每個項目的 `.accordion-header` 添加點擊事件監聽器。
- **預設展開**：在初始化時，檢查是否為第一個項目 (`index === 0`)，若是則將其設為展開狀態，並更新 ARIA 屬性和圖示。此邏輯應與 Jinja 模板的初始設定協同工作。
- **切換狀態**：
  - 點擊時，切換 `.active` class 來觸發 CSS 中定義的 `max-height` 和其他樣式變化。
  - 更新 `aria-expanded` 屬性。
  - 更新圖示文字 (`+` / `-`)。
- **`max-height` 設定**：
  - CSS 中 `.accordion-content.active` 已設定固定的 `max-height`。這是實現「固定高度內容」的主要方式。
  - 如果希望 `max-height` 根據內容動態調整（不適用於「固定高度」需求），可以取消註解 `content.style.maxHeight = content.scrollHeight + "px";` 這一行，並在摺疊時設為 `null`。
- **一次只展開一個項目 (可選)**：
  - 如果需要此行為 (典型的 Accordion)，請取消註解 `closeAllOtherItems(this);` 這一行。目前提供的程式碼（註解掉該行時）允許同時展開多個項目。

## 5. 資料整合 (從資料庫取得)

Accordion 的內容（標題、日期、詳細說明）將從資料庫獲取。後端應提供一個包含這些資料的列表或物件陣列。
前端 HTML 結構（如第 2 節所示）設計為易於透過伺服器端模板引擎（如 Jinja）或客戶端 JavaScript 框架進行資料填充。

**Jinja 範例 (後端資料結構示意):**
假設後端傳遞一個名為 `accordion_data` 的列表到模板，每個元素是一個包含 `title`, `date`, `content` 的字典：
```python
# Flask/Django 後端範例 (示意)
accordion_data = [
    {
        "id_suffix": "1", # 用於生成唯一 ID
        "title": "第一個標題",
        "date": "2023-10-26",
        "content": "這是第一個項目的詳細內容..."
    },
    {
        "id_suffix": "2",
        "title": "第二個標題，可能很長需要換行",
        "date": "2023-10-27",
        "content": "這是第二個項目的詳細內容..."
    }
    # ... 更多項目
]
# context = {'items': accordion_data} # 傳遞到模板 (Jinja 中使用 loop.index 或 item.id_suffix)
```
Jinja 模板中的渲染方式已在 HTML 結構一節中展示，可使用 `loop.index` 或後端提供的唯一 ID 來確保 `id` 和 `aria-` 屬性的唯一性。

## 6. 基本測試頁面

為了方便測試，以下是一個包含基本 HTML、CSS（內嵌）和 JavaScript（內嵌）的完整頁面。
您可以將此內容保存為一個 `.html` 檔案並在瀏覽器中打開進行測試。

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Accordion 測試頁面</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
      margin: 20px;
      background-color: #f4f4f4;
      color: #333;
    }
    .accordion-container {
      width: 100%;
      max-width: 700px;
      margin: 0 auto;
      border: 1px solid #ccc;
      border-radius: 5px;
      background-color: #fff;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    .accordion-item {
      border-bottom: 1px solid #eee;
    }
    .accordion-item:last-child {
      border-bottom: none;
    }
    .accordion-header {
      background-color: #f9f9f9;
      color: #333;
      cursor: pointer;
      padding: 15px 20px;
      width: 100%;
      text-align: left;
      border: none;
      outline: none;
      transition: background-color 0.2s ease;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 1.05em;
      font-weight: 500;
    }
    .accordion-header:hover,
    .accordion-header.active {
      background-color: #e7e7e7;
    }
    .accordion-title {
      flex-grow: 1;
      margin-right: 15px;
      line-height: 1.4;
    }
    .accordion-date {
      font-size: 0.85em;
      color: #555;
      margin-right: 15px;
      white-space: nowrap;
      flex-shrink: 0;
    }
    .accordion-icon {
      font-size: 1.2em;
      font-weight: bold;
      min-width: 20px;
      text-align: center;
      transition: transform 0.2s ease;
    }
    .accordion-content {
      padding: 0 20px;
      background-color: white;
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.3s ease-out, padding 0.3s ease-out;
      line-height: 1.6; /* 設定內容行高 */
    }
    .accordion-content.active {
      /* 固定高度：假設行高 1.6em，顯示 5 行，內容區 max-height 約 1.6em * 5 = 8em */
      /* 加上 padding-top 和 padding-bottom (各20px) */
      max-height: calc(8em + 40px); /* 範例固定高度，請依實際內容和字體大小調整 */
      overflow-y: auto; /* 如果內容超出此高度，則顯示滾動條 */
      padding: 20px;
      border-top: 1px solid #eee;
    }
    .accordion-content p {
      margin: 0 0 10px 0;
    }
    .accordion-content p:last-child {
      margin-bottom: 0;
    }

    /* RWD */
    @media (max-width: 600px) {
      .accordion-header {
        padding: 12px 15px;
        font-size: 1em;
      }
      .accordion-title {
        /* 允許標題在小螢幕上佔用更多空間，日期可能會被推到下一行或保持原樣 */
      }
      .accordion-date {
        font-size: 0.8em;
      }
      .accordion-content.active {
        /* 手機版可能需要不同高度或行數限制 */
        max-height: calc(10em + 40px); /* 例如，允許更多行 */
      }
    }
  </style>
</head>
<body>

<div class="accordion-container">
  <div class="accordion-item">
    <button id="accordion-header-ex1" class="accordion-header" aria-expanded="false" aria-controls="accordion-content-ex1">
      <span class="accordion-title">第一個標題：這是一個可能會非常長，需要換行顯示的標題範例</span>
      <span class="accordion-date">2024-01-15</span>
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
    <div id="accordion-content-ex1" class="accordion-content" role="region" aria-labelledby="accordion-header-ex1">
      <p>這是第一個項目的詳細內容。這部分內容將會有固定的高度限制。如果內容超過設定的高度，將會出現滾動條。</p>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.</p>
      <p>Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
      <p>Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo.</p>
      <p>Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt.</p>
    </div>
  </div>

  <div class="accordion-item">
    <button id="accordion-header-ex2" class="accordion-header" aria-expanded="false" aria-controls="accordion-content-ex2">
      <span class="accordion-title">第二個標題</span>
      <span class="accordion-date">2024-01-16</span>
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
    <div id="accordion-content-ex2" class="accordion-content" role="region" aria-labelledby="accordion-header-ex2">
      <p>這是第二個項目的詳細內容。</p>
      <p>較短的內容，不會觸發滾動條。</p>
    </div>
  </div>

  <div class="accordion-item">
    <button id="accordion-header-ex3" class="accordion-header" aria-expanded="false" aria-controls="accordion-content-ex3">
      <span class="accordion-title">第三個標題：只有一行</span>
      <span class="accordion-date">2024-01-17</span>
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
    <div id="accordion-content-ex3" class="accordion-content" role="region" aria-labelledby="accordion-header-ex3">
      <p>這是第三個項目的詳細內容。</p>
    </div>
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function () {
  const accordionItems = document.querySelectorAll('.accordion-item');

  function closeAllOtherItems(currentItemHeader) {
    accordionItems.forEach(item => {
      const header = item.querySelector('.accordion-header');
      if (header !== currentItemHeader && header.classList.contains('active')) {
        const content = item.querySelector('.accordion-content');
        const icon = header.querySelector('.accordion-icon');
        header.classList.remove('active');
        header.setAttribute('aria-expanded', 'false');
        content.classList.remove('active');
        if (icon) icon.textContent = '+';
      }
    });
  }

  accordionItems.forEach((item, index) => {
    const header = item.querySelector('.accordion-header');
    const content = item.querySelector('.accordion-content');
    const icon = header.querySelector('.accordion-icon');

    if (index === 0) { // 預設展開第一個
      header.classList.add('active');
      header.setAttribute('aria-expanded', 'true');
      content.classList.add('active');
      if (icon) icon.textContent = '-';
    } else {
      header.setAttribute('aria-expanded', 'false');
      if (icon) icon.textContent = '+';
    }

    header.addEventListener('click', function () {
      const isActive = this.classList.contains('active');
      
      // **取消註解下一行以啟用「一次只展開一個項目」的行為**
      // closeAllOtherItems(this); 

      if (isActive) {
        this.classList.remove('active');
        this.setAttribute('aria-expanded', 'false');
        content.classList.remove('active');
        if (icon) icon.textContent = '+';
      } else {
        this.classList.add('active');
        this.setAttribute('aria-expanded', 'true');
        content.classList.add('active');
        if (icon) icon.textContent = '-';
      }
    });
  });
});
</script>

</body>
</html>
```

## 7. 注意事項與未來擴展

- **Jinja 整合**：在將 HTML 結構套入 Jinja 模板時，請確保 `id`、`aria-controls`、`aria-labelledby` 等屬性的唯一性和對應關係，可利用 `loop.index` 或資料庫中的唯一 ID。
- **圖示替換**：目前的 `+` / `-` 圖示為純文字，可輕易替換為 SVG 圖示、圖示字型 (Font Awesome 等) 或 CSS 背景圖。
- **動畫效果**：CSS `transition` 提供了基本的滑動效果。可以根據需求調整動畫時間和效果曲線。
- **可訪問性 (Accessibility)**：已加入基本的 ARIA 屬性。請確保在實際應用中，鍵盤導航 (Tab, Enter/Space) 和螢幕閱讀器能正常運作。`<button>` 元素本身已提供部分支援。
- **內容固定高度**：CSS 中的 `.accordion-content.active` 的 `max-height` 屬性控制展開後內容區域的高度。請根據實際內容和設計需求（例如，固定顯示幾行文字，並考慮行高 `line-height`）來調整此值。如果希望內容超出時可滾動，請設定 `overflow-y: auto;`。
- **JavaScript 依賴**：此方案不依賴任何外部 JavaScript 函式庫 (如 jQuery)，使用原生 JavaScript 實現。

---

希望這份技術需求規格對您有所幫助！