```text

PROJECT_SPEC_BASIC.md

狀態：凍結基礎範圍
期間：11/3(宜起基) – 12/10(宜掛牌) 
目標：4 週做出「可上線、可 Demo、可放履歷」的購物網站（前台 + 後台 UI 基礎）

1. 專案概述：
做一個真的能用的購物網站：
訪客能逛商品、加入購物車、結帳（模擬付款）；
管理者能登入後台，新增/修改/刪除商品、上下架、調整庫存。
技術採用：
  前端（HTML, CSS, JavaScript）：React + Tailwind + figma。要試著使用TypeScript？
  後端(Java, SQL)：Spring Boot + PostgreSQL 
  技術展示：figma

  Java用21，我是用Oracle OpenJDK 21.0.8 – aarch64。
  用Spring Boot用3.5.7




2. 範圍：
前台：商品清單/詳情、購物車、結帳（模擬支付）、訂單列表、登入/註冊
後台 UI（Level 1）：登入、商品表格（分頁/基本排序）、新增/編輯/刪除、上下架、庫存±
後端：REST API（Swagger）、JWT、JPA、交易一致性（下單/庫存）
DB：PostgreSQL + Flyway V1（核心 5 張表）
部署：前端（Pages/Netlify）、後端（Render/Railway）

------ 還不確定要不要做------
圖片上傳（用 URL 欄位暫代）
進階查詢（多欄排序/分類/價格區間）
批次操作、審計日誌、CI/CD、自動化測試（整合/E2E）
------ 還不確定要不要做------

3. 使用者與情境：(技術亮點？)
訪客：我想瀏覽商品、搜尋關鍵字、看看詳情，再加入購物車。
會員：我想登入、把購物車結帳、付款（模擬）、之後能看到我的訂單。
管理者：我想登入後台、維護商品資料（CRUD）、上下架、調整庫存。

4. 主要頁面：
商品清單（分頁 + 搜尋欄）
/products/:id 商品詳情（名稱/價格/描述/主圖 URL、加入購物車）
/cart 購物車（改數量/刪除/去結帳）
/checkout 結帳（送出訂單、模擬付款 → 完成）
/orders 訂單列表與明細
/login、/register
/admin/login 後台登入
/admin/products 後台商品管理（表格 + Dialog 表單）
前端套件：React Router、React Hook Form + Zod、TanStack Query、Tailwind、（可選）shadcn/ui

5. API 概覽：
實際細節以 Swagger 為準；此處只列路徑與用途，方便前端對齊。
公開/會員
Method	Endpoint	說明
POST	/api/auth/register	註冊（BCrypt 雜湊）
POST	/api/auth/login	登入，回 JWT（含角色）
GET	/api/products?page=&size=&q=&sort=	商品清單（分頁 + 簡易搜尋）
GET	/api/products/{id}	商品詳情
GET	/api/cart	取得購物車（登入後）
POST	/api/cart/items	加入/更新購物車項目
DELETE	/api/cart/items/{itemId}	刪除購物車項目
POST	/api/orders	建立訂單（從購物車產生、扣庫存、狀態=PENDING）
POST	/api/payments/{orderId}/simulate	模擬付款 → 訂單改為 PAID
GET	/api/orders	我的訂單列表
GET	/api/orders/{id}	訂單明細
後台（需 ROLE_ADMIN）
Method	Endpoint	說明
GET	/api/admin/products?page=&size=&sort=	後台商品清單（分頁/基本排序）
POST	/api/admin/products	新增商品
PUT	/api/admin/products/{id}	編輯商品
DELETE	/api/admin/products/{id}	刪除商品
POST	/api/admin/products/{id}/publish	上/下架切換（body: { "published": true/false }）
POST	/api/admin/products/{id}/adjust-stock	庫存±（body: `{ "delta": 1

6. 資料模型：
Flyway V1 會建立以下表；圖片先用 URL 文字欄位。

users
id, email (unique), password_hash, role (USER/ADMIN), created_at

products
id, name, description, price_cents, stock, status (PUBLISHED/DRAFT), cover_image_url, created_at

orders
id, user_id (FK), total_cents, status (PENDING/PAID/CANCELLED), created_at

order_items
id, order_id (FK), product_id (FK), price_cents, qty

cart_items
id, user_id (FK), product_id (FK), qty

索引建議：products(name), products(price_cents), orders(user_id, created_at)

7. 非功能需求（NFR）：
安全：JWT + Spring Security；Admin 路由需 ROLE_ADMIN
驗證：前端（Zod）＋後端（Bean Validation）欄位規則一致
交易一致性：@Transactional 用於「生成訂單/扣庫存」「庫存調整」，禁止負庫存
文件：啟動即有 Swagger UI（OpenAPI）；README 列操作方式與 Demo 帳密
效能/可用性：所有清單均分頁；RWD 手機可用；錯誤有易懂訊息

8. 開發分工：
還沒想好

9. 進度表：

Week 1	專案骨架、DB V1、登入/註冊、商品清單/詳情

Week 2	購物車（本地 + 同步）、訂單建立（PENDING）

Week 3	模擬支付（PAID）、訂單列表、後台 CRUD（基本）

Week 4	上下架/庫存±、部署（前後端）、README/Swagger 完成

10. 驗收標準：
前台能完成「瀏覽 → 加入購物車 → 結帳（模擬）→ 查看訂單」全流程
後台可登入並完成商品新增/編輯/刪除、上下架、庫存±（錯誤訊息完善）
Swagger 可互動操作所有主要端點
部署完成且提供公開網址（前端＋後端）
README 含：架構圖、技術堆疊、啟動/部署方式、Demo 帳密

11. 可能的問題：
JWT/CORS 問題：預先設定允許來源；401 導回登入、403 顯示無權限
庫存負數：後端 Service 檢查（若 <0 回 422）；交易鎖定
分頁/排序不一致：統一使用 page,size,sort,q 命名
部署連線失敗：優先用同平台託管 DB；在 README 記錄環境變數

```
