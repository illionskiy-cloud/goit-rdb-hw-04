## Джерело датасету
**Brazilian E-Commerce Public Dataset by Olist** (Kaggle): <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>

## Спосіб отримання файлу
**KaggleHub.** У Colab має бути налаштований доступ до Kaggle (credentials / `KAGGLE_USERNAME` і `KAGGLE_KEY` у Secrets).

```python
import kagglehub
dataset_dir = kagglehub.dataset_download('olistbr/brazilian-ecommerce')
```

## Використані файли та таблиці

## Вхідні CSV

| CSV | Raw-таблиця | Typed-таблиця |
|---|---|---|
| `olist_customers_dataset.csv` | `olist_customers_raw` | `olist_customers` |
| `olist_orders_dataset.csv` | `olist_orders_raw` | `olist_orders` |
| `olist_order_items_dataset.csv` | `olist_order_items_raw` | `olist_order_items` |
| `olist_products_dataset.csv` | `olist_products_raw` | `olist_products` |
| `olist_sellers_dataset.csv` | `olist_sellers_raw` | `olist_sellers` |
| `olist_order_reviews_dataset.csv` | `olist_order_reviews_raw` | `olist_order_reviews` |

## Таблиці, створені в ході роботи

| Таблиця | Призначення | Завдання |
|---|---|---|
| `seller_alerts` | alert для продавця з найбільшою кількістю запізнень | 2.1, 2.4 |
| `seller_score` | агрегований рейтинг продавців (idempotent upsert) | 2.2 |
| `olist_orders.is_late` (нова колонка) | прапорець запізнення доставки | 2.3 |
| `olist_customer_dim` | дедуплікований dimension: один рядок на `customer_unique_id` | 3 |
| `customer_segments` | поточна сегментація покупців (VIP / regular / new / inactive) | 3 |

## Інструкція запуску

1. Відкрийте notebook у **Google Colab** (Python 3).
2. Підготуйте дані: налаштуйте Kaggle credentials (Варіант A) або покладіть CSV у `data/olist/` (Варіант B).
3. Виконайте першу клітинку: вона встановить залежності та запустить PostgreSQL:

   ```python
   %pip install -q pgserver psycopg2-binary sqlalchemy pandas kagglehub
   ```

   PostgreSQL піднімається локально через `pgserver` (каталог даних `/tmp/hw4_pg`), окремий сервер не потрібен.
4. Якщо в notebook використовується `%%sql`, підключіть jupysql до того самого сервера (приклад, адаптуйте під свій код):

   ```python
   %pip install -q jupysql
   %load_ext sql
   %config SqlMagic.displaylimit = 0   # показувати всі рядки результату
   %sql postgresql+psycopg2://... # URI з pg.get_uri()
   ```
5. Запускайте клітинки **строго зверху вниз**. Порядок важливий: raw → DROP/CREATE typed-таблиць → `INSERT ... SELECT` → DML → `olist_customer_dim` → `customer_segments` → запити.
6. Для повного перезапуску виберіть *Runtime → Restart and run all*. Усі DDL-блоки починаються з `DROP TABLE IF EXISTS`, а наповнення зроблене через `ON CONFLICT` та `UPDATE`, тому повторний запуск не створює дублікатів.

