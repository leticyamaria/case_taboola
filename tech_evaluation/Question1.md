# Question 1
## Using the JSON response from the below Recommendations API request, create a responsive Taboola widget with the following requirements.

### Widget requirements 
**For desktop and tablet:**   
- 3 columns and 2 rows (3x2).  
- The item thumbnail image aspect ratio should be 6:5.  
- Include the item title and branding below each image.  
- Add a header to the top of the widget with your choice of text.
- Clickable thumbnail image, item title, and branding text.  

**For smartphone**
- 1 column with six rows (1x6).
- The item thumbnail image aspect ratio should be 3:2.
- Include the item title and branding below each image.
- Add a header to the top of the widget with your choice of text.
- Clickable thumbnail image, item title, and branding text.
  
>[!IMPORTANT]
>You can only use pure HTML, CSS, and JavaScript to build and display the widget. Do not use any external libraries (e.g. jQuery, Bootstrap)

Please provide your solution via a shareable jsbin.com link that contains all of your working code

### LINK FOR JSBIN: [Taboola Widget](https://jsbin.com/mogeyisuyo/edit?html,css,js)  

**HTML**  
```HTML
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>

<section id="widget" aria-labelledby="widget-title">
  <h2 id="widget-title">You may like</h2>
  <span>Sponsored Links by Taboola</span/>
  <div class="grid" role="article_list"></div>
</section>

</body>
</html>
```
**CSS**  

```CSS
/* base */
* { box-sizing: border-box; }
body { 
  background: #F6F6F6;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
}

#widget {
  padding: 1rem;
  display: flex;
  flex-direction: column
}

span { 
  font-size: 0.8rem;
  text-align: right; 
  padding: 0.5rem
}

/* grid for mobile*/
.grid {
  display: grid;                
  grid-template-columns: 1fr;   
  gap: 0.625rem;                    
}

/* grid for desktop */
@media (min-width: 600px) {
  .grid { 
    grid-template-columns: repeat(3, 1fr); 
  }
}

/* card (inside grid) */
.card { 
  display: flex; 
  flex-direction: column; 
  gap: 0.5rem; 
  padding: 0.5rem; 
  border: 1px solid #e5e5e5; 
  background: #fff; 
}

/* link + image (inside card) */
.card a.thumb { 
  display: block; 
  width: 100%; 
  overflow: hidden; 
  background: #f3f3f3;  
}

.card a.thumb img {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
  aspect-ratio: 3 / 2;
}

@media (min-width: 600px) {
  .card a.thumb img { 
    aspect-ratio: 6 / 5; 
  }
}

/* title + brand (inside card) */
.card a.titulo {
  color: #111;
  text-decoration: none;
  font-weight: 600;
  line-height: 1.25;
  text-overflow: ellipsis;

/* ellipsis */
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card a.brand {
  color: #666;
  text-decoration: none;
  font-size: 0.9rem;
}
```
**JAVASCRIPT**
```javascript
// const =  familiar name
const API_URL = "https://api.taboola.com/1.2/json/apitestaccount/recommendations.get?app.type=web&app.apikey=7be65fc78e52c11727793f68b06d782cff9ede3c&source.id=%2Fdigiday-publishing-summit%2F&source.url=https%3A%2F%2Fblog.taboola.com%2Fdigiday-publishing-summit%2F&source.type=text&placement.organic-type=mix&placement.visible=true&placement.available=true&placement.rec-count=6&placement.name=Below%20Article%20Thumbnails&placement.thumbnail.width=640&placement.thumbnail.height=480&user.session=init";

const grid = document.querySelector(".grid");

// fetch api = goes to the api and fetch information
fetch(API_URL)
  .then(file => 
  file.json()) //transform into json
  .then(data => {
const items = data.list;
    items.forEach(item => { //create a list of attributes with json
      
// article = 1 instance of the grid      
const card = document.createElement("article");
      card.className = "card";
      card.setAttribute("role", "listitem");
// a = not visible      
const imagelink = document.createElement("a");
      imagelink.className = "thumb";
      imagelink.href = item.url;
      
const img = document.createElement("img");
      img.alt = item.name;
      img.src = item.thumbnail[0].url;
// inserts img link on img
      imagelink.appendChild(img);

const titlelink = document.createElement("a");
      titlelink.className = "titulo";
      titlelink.href = item.url;
      titlelink.target = "_blank";
      titlelink.textContent = item.name;

const brandlink = document.createElement("a");
      brandlink.className = "brand";
      brandlink.href = item.url;
      brandlink.target = "_blank";
      brandlink.textContent = item.branding;

      card.appendChild(imagelink);
      card.appendChild(titlelink);
      card.appendChild(brandlink);

      grid.appendChild(card);
    });
  }).catch((e) => {
  alert('Erro on fetch data')
})
```



