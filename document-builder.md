# Document Builder

`document-builder.html` la mot trang HTML thuần, tu chua toan bo giao dien, style va logic JavaScript de tao tai lieu dang block theo phong cach gan voi Notion khi xem tren web va khi in ra PDF.

Tai lieu nay duoc viet de co the:
- giai thich nhanh document builder dang hoat dong nhu the nao
- lam ban mo ta ky thuat de sau nay build lai mot phien ban tuong tu
- giup AI hoac nguoi phat trien doc lai va hieu cau truc ma khong can mo tung doan code

## 1. Muc tieu du an

Document Builder phuc vu viec tao mot trang soan thao tai lieu nhe, chay ngay trong trinh duyet, khong can backend va khong can build tool.

Muc tieu chinh:
- tao giao dien chinh sua tai lieu theo kieu block-based
- cho phep chuyen doi giua `Edit` va `Preview`
- cho phep keo tha block de sap xep lai thu tu
- luu trang thai vao `localStorage`
- co san sample report de sua nhanh
- bo cuc khi preview phu hop de in thanh PDF dep va de doc

## 2. Nguyen ly kien truc

Toan bo du an nam trong mot file duy nhat: `document-builder.html`.

Cau truc file gom 3 phan lon:

### 2.1. HTML structure

Layout tong the chia thanh 2 cot:
- `aside.sidebar`: bang cong cu ben trai, co search va 3 nhom block (`Favorite 1`, `Favorite 2`, `All blocks`)
- `section.workspace`: khu vuc chinh sua va xem truoc tai lieu

Trong workspace co:
- `topbar`: tieu de tai lieu, thong tin save, nut chuyen mode
- `canvas-shell`: khung chua tai lieu
- `paper`: mat giay gia lap de tao cam giac tai lieu khi preview/in PDF
- `blockList`: noi render danh sach block hien tai

### 2.2. CSS styling

CSS duoc viet inline trong the `<style>` va chia theo nhom:
- bien mau trong `:root`
- layout tong the: `body`, `.app`, `.sidebar`, `.workspace`
- thanh cong cu, card block, nut, pill
- khu vuc giay tai lieu `.paper`
- style rieng cho tung loai block khi render
- responsive cho tablet/mobile

Huong thiet ke hien tai:
- nen am, giay sang, bo cuc editorial nhe
- chu dao sans-serif de de doc va can doi hon
- heading ro cap bac nhung khong qua nang
- bo goc mem, border mong, shadow nhe
- preview sach hon edit mode

### 2.3. JavaScript behavior

JavaScript thuần quan ly state va render lai UI.

State chinh:
- `blocks`: mang chua toan bo block trong tai lieu
- `globalMode`: `edit` hoac `preview`
- `dragging`: luu thong tin block dang keo tha
- `toastTimer`: quan ly thong bao save nho

Hang so quan trong:
- `STORAGE_KEY = 'document_builder_state_v4'`
- `stickyColors`: bang mau cho sticky note

## 3. Model du lieu block

Moi block co dang co ban:

```js
{
  id: 'blk_xxx',
  type: 'text|heading|bulleted|todo|quote|callout|columns|image|sticky|divider',
  mode: 'edit'
}
```

Mot so block co them thuoc tinh rieng:

- `heading`: `level`, `text`
- `text`: `html`
- `bulleted` / `todo`: `itemsText`
- `quote`: `text`
- `callout`: `tone`, `text`
- `columns`: `count`, `columns`
- `image`: `src`, `alt`
- `sticky`: `text`, `color`

Ham `defaultBlock(type)` la noi khoi tao du lieu mac dinh cho tung block.

## 4. Danh sach block hien co

Palette ben trai dang ho tro 13 loai block, duoc to chuc thanh:

- `Favorite 1`: nhom block dung nhieu nhat, co the keo tha de doi thu tu
- `Favorite 2`: nhom block uu tien thu hai, co the keo tha de doi thu tu
- `All blocks`: phan con lai, trong do heading duoc gom chung de quet nhanh hon

Sidebar cung co o `search` de loc block theo ten, mo ta hoac `type`.

Danh sach block hien co:

1. `text` - doan van ban rich text
2. `heading1` - heading cap 1
3. `heading2` - heading cap 2
4. `heading3` - heading cap 3
5. `bulleted` - danh sach bullet
6. `todo` - danh sach checklist
7. `quote` - trich dan/insight
8. `callout` - hop thong diep quan trong
9. `columns2` - layout 2 cot
10. `columns3` - layout 3 cot
11. `image` - hinh anh tu URL hoac upload file
12. `sticky` - ghi chu nhanh
13. `divider` - duong phan tach noi dung

Luu y: `heading1`, `heading2`, `heading3` khi dua vao state deu tro thanh `type: 'heading'` voi `level` khac nhau.

## 5. Luong hoat dong chinh

### 5.1. Khoi tao trang

Khi mo file:
- ham `load()` doc du lieu tu `localStorage`
- neu chua co du lieu thi goi `loadSample()`
- sau do goi `render()` de ve lai giao dien

### 5.2. Them block

Nguoi dung co the them block bang 2 cach:
- bam vao block trong palette
- keo block tu palette vao canvas

Ngoai ra, block trong 2 nhom favorite co the duoc keo tha qua lai giua `Favorite 1`, `Favorite 2` va `All blocks` de user tu sap lai menu cho de dung.

Ham su dung:
- `addBlock(type, index)`
- `defaultBlock(type)`

### 5.3. Sua noi dung

Moi block co UI rieng de sua:
- heading dung `input`
- text dung `contenteditable`
- list/todo/quote/callout dung `textarea`
- image ho tro URL va upload file bang `FileReader`
- columns cho phep sua tung cot rieng
- sticky cho phep doi mau

Cap nhat state chu yeu qua:
- `updateBlock(id, patch, rerender)`
- `updateColumn(id, index, value)`

### 5.4. Preview mode

Ham `setMode(mode)` chuyen giua `Edit` va `Preview`.

Trong `Preview`:
- an meta tool va nut dieu khien block
- bo border card de tai lieu sach hon
- hien thi noi dung theo style tai lieu thay vi form input

### 5.5. Sap xep block bang drag and drop

Drag and drop duoc ho tro cho:
- keo tu palette vao canvas de them moi
- keo block dang co de doi thu tu

Ham lien quan:
- `handleCanvasDragOver(event)`
- `getDropIndex(event)`
- `showDropLine(index)`
- `clearDropLine()`

### 5.6. Luu trang thai

Moi lan thay doi noi dung, he thong se goi `save(message, showToastMessage)`.

Thong tin duoc luu:
- `title`
- `blocks`

Noi luu:
- `localStorage` cua trinh duyet

Y nghia:
- mo lai file van giu nguyen noi dung da sua tren cung mot may/trinh duyet

## 6. Cac ham quan trong can nho

### State va khoi tao
- `uid()` - tao id cho block
- `defaultBlock(type)` - tao block mac dinh
- `load()` - nap state da luu
- `loadSample()` - nap sample report co san
- `save(message, showToastMessage)` - luu state

### Thao tac block
- `addBlock(type, index)` - them block
- `addBelow(id)` - them 1 text block ben duoi block hien tai
- `duplicateBlock(id)` - nhan doi block
- `deleteBlock(id)` - xoa block
- `clearCanvas()` - xoa toan bo canvas

### Render
- `render()` - render toan bo danh sach block
- `renderBlock(block, index)` - render 1 block
- `renderMeta(block)` - render meta badge/select cho block
- `renderContent(block)` - dieu huong sang renderer phu hop

### Render theo loai block
- `renderHeading()`
- `renderText()`
- `renderList()`
- `renderTodo()`
- `renderQuote()`
- `renderCallout()`
- `renderColumns()`
- `renderImage()`
- `renderSticky()`

### Utility
- `changeColumnCount(id, value)` - doi so cot
- `setImageUrl(id, input)` - gan URL anh
- `uploadImage(id, file)` - doc file image local
- `formatText(id, command)` - rich text bang `document.execCommand`
- `escapeHtml(value)` / `escapeAttr(value)` - chong vo HTML
- `filterPalette(query)` - loc block trong sidebar
- `setupPalette()` / `attachPaletteDnD()` - chia section va gan drag-drop cho palette

## 7. Sample report dang co

Ham `loadSample()` dang dung mot bo du lieu mau la bao cao kinh doanh 6 thang dau nam 2019.

Muc dich cua sample:
- trinh dien ngay bo cuc giong mot tai lieu that
- giup user sua nhanh ma khong phai tao tu trang trong
- cho thay cach ket hop heading, text, callout, quote, columns, checklist

Neu build lai he thong khac, co the tach sample nay thanh JSON rieng.

## 8. Dinh huong UI/UX hien tai

Document Builder nay khong copy 1:1 Notion, ma lay cam hung tu Notion export PDF:

- heading lon nhung co kiem soat
- than bai doc thoang, do dam vua phai
- phan cap thong tin ro rang
- co cam giac trang giay that trong khung xem
- edit mode than thien, preview mode sach

Mot so diem phong cach dang duoc ap dung:
- sidebar mo ta block dang card
- paper canvas co duong line doc ben trai tao cam giac editorial
- callout co tone `info`, `warning`, `success`
- quote dung border-left me mai
- columns dung de tom tat va so sanh nhanh

## 9. Nhung diem manh cua phien ban hien tai

- chay doc lap, mo file HTML la dung duoc
- khong phu thuoc framework
- de sua nhanh vi HTML/CSS/JS nam chung 1 cho
- co local persistence
- de in PDF tu trinh duyet
- co drag and drop co ban
- co sample thuc te de demo

## 10. Han che hien tai

Day la nhung han che can nho neu build lai:

- khong co backend, khong dong bo du lieu giua may
- `document.execCommand` la API cu, co the thay bang rich text editor hien dai hon neu nang cap
- khong co pagination that su khi in PDF
- chua co export/import JSON ro rang
- chua co block nesting phuc tap nhu toggle database, table, synced block that
- chua co co che undo/redo
- chua co print stylesheet rieng cho A4

## 11. Goi y neu muon build lai ban tot hon

Neu sau nay build lai mot phien ban tuong tu, nen tach thanh cac module sau:

### 11.1. Data layer
- `blockTypes.ts` hoac `blockTypes.js`
- `sampleDocument.json`
- `storage.js`

### 11.2. Render layer
- renderer rieng cho moi block
- template system hoac component nhe

### 11.3. Interaction layer
- drag and drop module
- rich text module
- keyboard shortcuts
- undo/redo stack

### 11.4. Export layer
- print CSS cho A4
- export JSON/import JSON
- co the them export HTML/PDF sau nay

## 12. Blueprint de rebuild nhanh

Neu can lam lai tu dau, co the theo thu tu nay:

1. Tao layout 2 cot: sidebar + workspace
2. Tao state `blocks[]` va `globalMode`
3. Tao `defaultBlock(type)`
4. Tao `render()` va `renderBlock()`
5. Ho tro 4 block co ban truoc: heading, text, bullet, divider
6. Them save/load bang `localStorage`
7. Them preview mode
8. Them drag and drop
9. Them callout, columns, image, sticky
10. Chinh CSS cho cam giac giay va kha nang in PDF

## 13. Cach AI nen doc file nay trong tuong lai

Neu sau nay can yeu cau AI chinh sua `document-builder.html`, nen doc theo thu tu:

1. Doc phan `Muc tieu du an`
2. Doc `Model du lieu block`
3. Doc `Luong hoat dong chinh`
4. Xac dinh block nao can sua
5. Sau do moi vao file HTML de sua dung ham va dung class lien quan

Dieu nay giup tranh sua nham vi file hien tai la file don, kha dai va chua ca UI lan logic.

## 14. File lien quan

- `document-builder.html`: file chinh cua project
- `document-builder.md`: tai lieu mo ta nay

## 15. Tom tat ngan

Day la mot block-based document editor mini, chay bang HTML/CSS/JS thuần, huong toi viec tao tai lieu dep de xem truoc va in PDF theo tinh than Notion. File hien tai phu hop de demo, chinh sua nhanh, va lam nen tang cho mot phien ban lon hon trong tuong lai.