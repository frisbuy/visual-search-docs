# API визуального поиска

# Общие сведения
## Требования к файлам изображений
- формат изображений: jpeg или png
- разрешение:
  - оптимально: 1000 точек по большей из сторон
  - приемлемо: 800 точек по большей из сторон (снижение качества поиска незначительно)
  - минимально приемлемо: 640 точек по большей из сторон (снижение качества поиска можно заметить, но результат в среднем приемлемый)
  - увеличение разрешения свыше 1200 по большей стороне не улучшает качество поиска, но увеличивает время обработки
- размер файла:
  Строгих ограничений нет; при оптимальном разрешении (1000 точек по большей стороне) размер файла в формате jpeg обычно составляет порядка 100 кб. При этом скорость и качество поиска оптимальны. Если использовать файлы большего размера - всё должно работать, но займёт больше времени. Файлы до 10 мб должны обрабатываться корректно.
- качество снимка влияет на качество поиска; ухудшить могут такие факторы, как
  - большое количество шума
  - очень плохое освещение
  - сильно пересвеченное фото
  - низкий контраст предметов одежды с фоном (к примеру, чёрное платье на фоне чёрной стены)
  - сильно размазанный снимок (нет фокуса, или изображение снято в движении); небольше размытие не скажется
# Методы API
## Метод Upload

**Описание**
Используется для прямой загрузки файла с изображением

- url: https://api.frisbuy.com/api/visual_search/v1/upload
- метод: POST
- enctype: multipart/form-data
- параметры передаются либо в POST, либо в query; можно сочетать оба способа, и query имеет приоритет над POST
  - `api_token`: токен пользователя (string, 32 байта) - выдаётся аккаунт-менеджером, определяет, среди каких товаров будет происходи поиск (т.е. какой товарный фид будет использован)
  - `image` (файл) - изображение, загруженное пользователем
  - `upload``_o``nly` - режим предварительной загрузки фото
  - `gender` (пол) - пол (string, ‘male’|’female’| undefined or null (все))
  - `request_id`: id запроса (MongoDB ObjectID, 24 байта, опционально; использутся для двухфазного поиска, в сочетании

**Пример использования**

    curl -F 'image=@Kozhanoe-ispolnenie.jpg' 'https://api.frisbuy.com/api/visual_search/v1/upload?api_token=kpX2xkqVWUkH&gender=female'


## Метод HTTP Get

**Описание**
Используется для скачивания файлов изображений по прямой HTTP ссылке; также поддерживает ссылки на посты в инстаграм.

- url: https://api.frisbuy.com/api/visual_search/v1.2.0/http-get
- метод: POST
- enctype: multipart/form-data
- параметры передаются либо в POST, либо в query; можно сочетать оба способа, и query имеет приоритет над POST
  - `api_token`: токен пользователя (string, 32 байта) - выдаётся аккаунт-менеджером, определяет, среди каких товаров будет происходи поиск (т.е. какой товарный фид будет использован)
  - `url`  - ссылка на файл изображения или на пост в Инстаграм
  - `upload``_o``nly` - режим предварительной загрузки фото
  - `gender` (пол) - пол (string, ‘male’|’female’| undefined or null (все))
  - `request_id`: id запроса (MongoDB ObjectID, 24 байта, опционально; использутся для двухфазного поиска, в сочетании

**Примеры использования**
Прямая ссылка на файл:

    curl 'https://api.frisbuy.com/api/visual_search/v1/http-get?api_token=kpX2xkqVWUkH&url=https%3A%2F%2Fscontent-frt3-1.cdninstagram.com%2Fvp%2F0d3ee05020651bfa20b0feb4718d7883%2F5D0BB461%2Ft51.2885-15%2Fe35%2F51962016_480073726094227_1289207511322387173_n.jpg%3F_nc_ht%3Dscontent-frt3-1.cdninstagram.com&gender=female'

Пост из Instagram:

    curl 'https://api.frisbuy.com/api/visual_search/v1/http-get?api_token=kpX2xkqVWUkH&url=https%3A%2F%2Fwww.instagram.com%2Fp%2FBuYcno5FV74%2F&gender=female'


# Двухфазный поиск

Параметр `upload_only` позволяет выполнить только загрузку изображения, но не выполнять поиск. Это может быть полезно, к примеру, в случае загрузки публикации из инстаграм, чтобы показать превью изображения до того, как будет выполнен поиск товаров.

При этом, в первой фазе (загрузка изображения) достаточно будет передать только токен апи, само изображение, и параметр `upload_only`:

    curl 'https://api.frisbuy.com/api/visual_search/v1/http-get?api_token=kpX2xkqVWUkH&url=https%3A%2F%2Fwww.instagram.com%2Fp%2FBuYcno5FV74%2F&upload_only=1'

Вернётся сокращённый ответ, содержащий только ссылки на изображения, и ID запроса, который понадобится для второй фазы:

    {
      "response": {
        "image": {
          "original": "https://s3.us-east-2.amazonaws.com/visual-search.s3.frisbuy.com/5c766811b489c5027b7bd4c1",
          "medium": "https://s3.us-east-2.amazonaws.com/visual-search.s3.frisbuy.com/5c766811b489c5027b7bd4c1",
          "thumbnail": "https://s3.us-east-2.amazonaws.com/visual-search.s3.frisbuy.com/5c766811b489c5027b7bd4c1"
        }
      },
      "request": {
        "id": "5c76744bb489c5027d48c673", // ID запроса
        "success": true,
        "errors": []
      }
    }

Для второй фазы (поиск товаров) изображение передавать уже не нужно, достаточно указать только request_id, и дополнительные параметры поиска (такие, как gender):

    curl 'https://api.frisbuy.com/api/visual_search/v1/http-get?api_token=kpX2xkqVWUkH&request_id=5c76744bb489c5027d48c673&gender=female'
# Формат ответа

В теле ответа приходит JSON следующей структуры: 

    {
      "response": {
        "image": {
            "original": "", // url, оригинальное изображение, загруженное пользователем
            "medium": "https://...", // url, 600x600 или меньше
            "thumbnail": "https://...", // url, 300x300 или меньше
        },
        "productPredictionList": [
          {}, // объект ProductPrediction
          {}, // объект ProductPrediction
          ...
        ],
      },
      "request": {
        "id": "507f1f77bcf86cd799439011"
        "success": true, // бывает false, если в errors не пусто
        // для текущей версии API - в errors всегда будет либо 0 эл-тов, либо ровно 1
        "erorrs": [
          {
            // см. перечень кодов ошибок ниже
            "code": "photo.no_clothes_found",
            "text": "На фото не найдено ни одного предмета одежды",
          }
        ]
      }
    }

**ProductPrediction**

    {
      // код класса одежды, перечень кодов поставим
      "clothesClassCode": <string>,
      "clothesClassName": <string>,
      "bbox": <Bbox> // Объект типа Bbox
      "goods": [...], // массив объектов типа Good
    }

**Good**

    {
      // из product feed
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

**Bbox**

    [x1,y1,x2,y2] 
    // Координаты л.в. и п.н. углов прямоугольника, описывающего найденный на фото объект; в процентах ширины и высоты, от левого верхнего угла
# Коды ошибок
- request.api_token.missing:        Не передан токен доступа для API
- request.api_token.invalid:        Некорректный доступа для API
- request.image.empty:        Изображение не было загружено
- request.url.empty:        Адрес изображения не передан
- instagram.private_account:        Изображение находится в приватном аккаунте.\nПопробуйте сохранить на свой компьютер и загрузить как файл
- instagram.unknown_error:        Не удалось скачать изображение из Instagram,\nпопробуйте сохранить на свой компьютер и загрузить как файл
- request.http.timeout:        Не удалось скачать изображение со стороннего сайта,\nпопробуйте сохранить на свой компьютер и загрузить как файл
- request.url.invalid:        Некорректный адрес изображения. Перепроверьте адрес,\nили попробуйте сохранить изображение на свой компьютер и загрузить как файл
- request.http.code:        Не удалось скачать изображение со стороннего сайта,\nпопробуйте сохранить на свой компьютер и загрузить как файл
- request.image.format_invalid:        К сожалению, этот формат изображения не поддерживается.\nПожалуйста, убедитесь, что вы загрузили правильный файл.
- error.internal.storage:        Произошёл сбой на нашей стороне. Пожалуйста, попробуйте ещё раз.
- error.internal.ml:        Произошёл сбой на нашей стороне. Пожалуйста, попробуйте ещё раз.
- error.internal.ml.empty_response:        Произошёл сбой на нашей стороне. Пожалуйста, попробуйте ещё раз.
# Коды классов одежды
| ClothesClassId | ClothesClassCode | ClothesClassName | Transliteration |
| -------------- | ---------------- | ---------------- | --------------- |
| 0              | blouse           | блуза            | bluzka          |
| 1              | high_shoes       | ботинки          | botinki         |
| 2              | trousers         | брюки            | bryuki          |
| 3              | t_shirt          | футболка         | futbolka        |
| 4              | headdress        | головной убор    | golovnoj_ubor   |
| 5              | gumshoes         | кеды             | kedy            |
| 6              | sweater          | кофта            | kofta           |
| 7              | tights           | колготки         | kolgotki        |
| 8              | snickers         | кроссовки        | krossovki       |
| 9              | swimsuit         | купальник        | kupal'nik       |
| 10             | jacket           | куртка           | kurtka          |
| 11             | brassiere        | лиф              | lifchik         |
| 12             | coat             | пальто           | pal'to          |
| 13             | blazer           | пиджак           | pidzhak         |
| 14             | cloak            | плащ             | plashch         |
| 15             | dress            | платье           | plat'e          |
| 16             | shirt            | рубашка          | rubashka        |
| 17             | sandals          | сандали          | sandali         |
| 18             | boots            | сапоги           | sapogi          |
| 19             | shorts           | шорты            | shorty          |
| 20             | fur_coat         | шуба             | shuba           |
| 21             | handbag          | сумка            | sumka           |
| 22             | panties          | трусы            | trusy           |
| 23             | low_shoes        | туфли            | tufli           |
| 24             | skirt            | юбка             | yubka           |

