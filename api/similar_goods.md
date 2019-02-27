# API визуального поиска

# Методы API
## Метод Similar Goods

**Описание**

Выполняет подбор товаров, похожих на указанный


- url: https://api.frisbuy.com/api/visual_search/v1/similar-goods
- метод: GET
- параметры передаются в query
  - `api_token`: токен пользователя (string, 32 байта) - выдаётся аккаунт-менеджером, определяет, среди каких товаров будет происходи поиск (т.е. какой товарный фид будет использован)
  - `sku` - id товара (SKU), соответствует offerId в product feed
  - `group` - id группы товаров, соответствует offerId в product feed

Достаточно передать любой из двух параметров, `sku` или `group`.

**Формат ответа**

    {
      "response": {
        "goods": [
          {}, // объект типа Good
        ]
      }
    }


**Good**

    {
      "offerId": <string>, // значение Offer Id из Product Feed клиента
      "groupId": <string>, // значение Group Id из Product Feed клиента
      "vendorCode" <string>, // артикул
      "name": <string>, // название товара
      "price": <int>, // актуальная цена
      "priceOld": <int>, // старая цена (если действует скидка, иначе будет null
      "brandId": <string>, // Название бренда
      "description": <string>, // описание товара
      "vendorShopUrl": <string> // ссылка на товар в магазине
      "thumbUrl": <string>, // ссылка на preview, в CDN Frisbuy, размер - не более 300x300
      "confidence": float, // степень похожести товара на то, что на фото, от 0 до 100
    }
