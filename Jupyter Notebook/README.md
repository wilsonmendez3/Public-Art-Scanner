
### Collect data from API endpoint, curate it and add it to a dataframe


```python
# Import Dependencies
import json 
import pprint 
import pandas as pd
import requests

from bs4 import BeautifulSoup as bs
from IPython.display import Image, HTML, display
```


```python
# Pull the 'data' from the API endpoint and display it
base_url = 'https://openaccess-api.clevelandart.org/api/artworks/?q=nude'    
clevelandart_data = requests.get(base_url).json()
clata = clevelandart_data["data"]
# pprint.pprint(clata)
```


```python
# Pull the following information for each piece of art: 
# Title, Artist, Creation Date, Description, Image URLS
cleve_list = []
locations = []

for art in clata:
    cleve_d = {}
    cleve_d["title"] = art["title"]
    artist = art["creators"]
    for item in artist:
        cleve_d["artist"] = item["description"]
    cleve_d["creation_date"] = art["creation_date"]
    location_list = art["culture"]
    for loc in location_list:
        locations.append(loc)
    for loc in locations:
        cleve_d["location"] = loc
    cleve_d["digital_description"] = art["digital_description"]
    cleve_d["fun_fact"] = art["fun_fact"]
    cleve_d["wall_description"] = art["wall_description"]
    cleve_d["post_url"] = art["url"]
    cleve_list.append(cleve_d)
        
# pprint.pprint(cleve_list)
```


```python
# Convert the curated data into a dataframe and display it
cleve_df = pd.DataFrame(data=cleve_list)
pd.set_option('display.max_colwidth', -1)

# Split location column to grab just the country and drop the location column
country = cleve_df["location"].str.split(",", n = 1, expand = True)
cleve_df["country"] = country[0]
cleve_df.drop(['location'], axis=1)

columnsTitles = ['title', 'artist', 'creation_date', 'country', 'digital_description', 'fun_fact', 'wall_description', 'post_url',]

cleve_df = cleve_df.reindex(columns=columnsTitles)

cleve_df.head()
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>artist</th>
      <th>creation_date</th>
      <th>country</th>
      <th>digital_description</th>
      <th>fun_fact</th>
      <th>wall_description</th>
      <th>post_url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Reclining Nude (Fernande)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1906</td>
      <td>Spain</td>
      <td>This drawing depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Picasso was engaged in radical experimentation with technique and simplified formal distortions at this time, and this shift in his work can be seen in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>Picasso mixed his paints with an unidentified material to produce the spotted effect most noticeable on Fernande's lower torso and the blue blanket below her.</td>
      <td>Reclining Nude depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Radical experimentation with technique and simplified formal distortions characterize Picasso's work at this time and can be seen here, both in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>https://clevelandart.org/art/1954.865</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nude</td>
      <td>Roger Parry (French, 1905-1977)</td>
      <td>1930</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2007.102</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nude</td>
      <td>John Bernard Flannagan (American, 1895-1942)</td>
      <td>1941</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.161</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Nude</td>
      <td>James Newberry (American, 1937-)</td>
      <td>c. 1974</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.325</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nude</td>
      <td>Brassaï (French, 1899-1984)</td>
      <td>c. 1931-1933</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Solarization is a partial reversal of tones in an image caused by re-exposing a negative or positive to light during the development process. Like many of the techniques explored by Surrealist photographers, it distances the photograph from the factual recording of reality. Originally discovered in 1862 by Armand Sabattier, it was rediscovered around 1930 by Lee Miller and Man Ray (Man Ray’s portrait of Miller is in the Avant-Garde Portrait section of the exhibition).</td>
      <td>https://clevelandart.org/art/2008.174</td>
    </tr>
  </tbody>
</table>
</div>



### Scrape the Image URLs and add them to a list


```python
# Convert "post_url" column to a list
post_urls = cleve_df["post_url"].tolist()
# pprint.pprint(post_urls)
```


```python
# Scrape the image URLs by looping through each image URL
img_urls = []
for url in post_urls:
    response = requests.get(url)
    soup = bs(response.text, 'lxml')

    img_src = soup.find("img", {"typeof":"foaf:Image"})
    if img_src != None:
        img_urls.append(img_src['src'])
    else:
        img_urls.append("Sorry! No URL available.")
```


```python
# Add img_urls as a column to the dataframe
cleve_df["img_url"] = img_urls
cleve_df.head()
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>artist</th>
      <th>creation_date</th>
      <th>country</th>
      <th>digital_description</th>
      <th>fun_fact</th>
      <th>wall_description</th>
      <th>post_url</th>
      <th>img_url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Reclining Nude (Fernande)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1906</td>
      <td>Spain</td>
      <td>This drawing depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Picasso was engaged in radical experimentation with technique and simplified formal distortions at this time, and this shift in his work can be seen in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>Picasso mixed his paints with an unidentified material to produce the spotted effect most noticeable on Fernande's lower torso and the blue blanket below her.</td>
      <td>Reclining Nude depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Radical experimentation with technique and simplified formal distortions characterize Picasso's work at this time and can be seen here, both in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>https://clevelandart.org/art/1954.865</td>
      <td>https://piction.clevelandart.org/cma/ump.di?e=91AEDF5376158FC04BC845A0D8CB7D486ACCAF4A2D77B46FB298C305C0368FEE&amp;s=21&amp;se=1031267696&amp;v=&amp;f=1954.865_o10.jpg</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nude</td>
      <td>Roger Parry (French, 1905-1977)</td>
      <td>1930</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2007.102</td>
      <td>https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF0B3993F1E232850427E9D8AD9710DB47959C2928AC3866F5&amp;s=21&amp;se=1828349793&amp;v=&amp;f=2007.102_o10.jpg</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nude</td>
      <td>John Bernard Flannagan (American, 1895-1942)</td>
      <td>1941</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.161</td>
      <td>https://piction.clevelandart.org/cma/ump.di?e=5B79BA773164218692DF8DE5386094B3570F33D196F4E232566CEF1C28EB7810&amp;s=21&amp;se=1674154316&amp;v=&amp;f=1961.161_o10.jpg</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Nude</td>
      <td>James Newberry (American, 1937-)</td>
      <td>c. 1974</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.325</td>
      <td>https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762B0BC9035C4AB4AFD5E4EB11973141B8E8C61CF7F86997072E&amp;s=21&amp;se=525383189&amp;v=3&amp;f=1992.325_o10.jpg</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nude</td>
      <td>Brassaï (French, 1899-1984)</td>
      <td>c. 1931-1933</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Solarization is a partial reversal of tones in an image caused by re-exposing a negative or positive to light during the development process. Like many of the techniques explored by Surrealist photographers, it distances the photograph from the factual recording of reality. Originally discovered in 1862 by Armand Sabattier, it was rediscovered around 1930 by Lee Miller and Man Ray (Man Ray’s portrait of Miller is in the Avant-Garde Portrait section of the exhibition).</td>
      <td>https://clevelandart.org/art/2008.174</td>
      <td>https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF6479C2A7B7E2E3AC60AD8C4CA4DA37AB7455FAAF81526B7B&amp;s=21&amp;se=1828349793&amp;v=&amp;f=2008.174_o10.jpg</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Define a function to add images to the dataframe
def img_to_df(path):
   '''
    This function essentially convert the image url to
    '<img src="'+ path + '"/>' format. And one can put any
    formatting adjustments to control the height, aspect ratio, size etc.
    within as in the below example.
   '''

   return '<img src="'+ path + '" style=max-width:250px;"/>'

HTML(cleve_df.to_html(escape=False, formatters=dict(img_url=img_to_df)))
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>artist</th>
      <th>creation_date</th>
      <th>country</th>
      <th>digital_description</th>
      <th>fun_fact</th>
      <th>wall_description</th>
      <th>post_url</th>
      <th>img_url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Reclining Nude (Fernande)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1906</td>
      <td>Spain</td>
      <td>This drawing depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Picasso was engaged in radical experimentation with technique and simplified formal distortions at this time, and this shift in his work can be seen in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>Picasso mixed his paints with an unidentified material to produce the spotted effect most noticeable on Fernande's lower torso and the blue blanket below her.</td>
      <td>Reclining Nude depicts Picasso's lover Fernande Olivier, with whom he spent the summer of 1906 in the remote Spanish village of Gósol, where he made this sheet. Radical experimentation with technique and simplified formal distortions characterize Picasso's work at this time and can be seen here, both in Fernande's stylized face and somewhat disjointed gesture, and in the expressive strokes and spotted effects of the gouache.</td>
      <td>https://clevelandart.org/art/1954.865</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=91AEDF5376158FC04BC845A0D8CB7D486ACCAF4A2D77B46FB298C305C0368FEE&s=21&se=1031267696&v=&f=1954.865_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nude</td>
      <td>Roger Parry (French, 1905-1977)</td>
      <td>1930</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2007.102</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF0B3993F1E232850427E9D8AD9710DB47959C2928AC3866F5&s=21&se=1828349793&v=&f=2007.102_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nude</td>
      <td>John Bernard Flannagan (American, 1895-1942)</td>
      <td>1941</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.161</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA773164218692DF8DE5386094B3570F33D196F4E232566CEF1C28EB7810&s=21&se=1674154316&v=&f=1961.161_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Nude</td>
      <td>James Newberry (American, 1937-)</td>
      <td>c. 1974</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.325</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762B0BC9035C4AB4AFD5E4EB11973141B8E8C61CF7F86997072E&s=21&se=525383189&v=3&f=1992.325_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nude</td>
      <td>Brassaï (French, 1899-1984)</td>
      <td>c. 1931-1933</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Solarization is a partial reversal of tones in an image caused by re-exposing a negative or positive to light during the development process. Like many of the techniques explored by Surrealist photographers, it distances the photograph from the factual recording of reality. Originally discovered in 1862 by Armand Sabattier, it was rediscovered around 1930 by Lee Miller and Man Ray (Man Ray’s portrait of Miller is in the Avant-Garde Portrait section of the exhibition).</td>
      <td>https://clevelandart.org/art/2008.174</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF6479C2A7B7E2E3AC60AD8C4CA4DA37AB7455FAAF81526B7B&s=21&se=1828349793&v=&f=2008.174_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>5</th>
      <td>Nude</td>
      <td>William K. Leuthold (American)</td>
      <td>c. 1929</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1929.432</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C10A65F951668E75FF5B03E59D729207B3CE5F3FE341EDA39&s=21&se=1049999129&v=&f=1929.432_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>6</th>
      <td>Nude</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.260</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E0CAE4E9429C24FB3DCFF417BABE65E686D2AFCDEF0BD04AC&s=21&se=1031267696&v=3&f=1941.260_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>7</th>
      <td>Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>before 1915</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.805</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C5745ECCC2E20DCCB1ED8C5CC81374D2B9E689260E881B57FA8&s=21&se=1031267696&v=6&f=1954.805_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>8</th>
      <td>Nude</td>
      <td>James Newberry (American, 1937-)</td>
      <td>c. 1973</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.324</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762BEBAF7BD531B99C39F1F0847F537702B36A8C7EF1E0043BCE&s=21&se=525383189&v=3&f=1992.324_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>9</th>
      <td>Nude</td>
      <td>Emil Ganso (American, 1895-1941)</td>
      <td>1932</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1947.140</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9FD37BAFC27E9FE14E3C807BCCC026EB9FD583E3A80A108B63EF695D7E0E7814&s=21&se=1031267696&v=&f=1947.140_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>10</th>
      <td>Seated Male Nude</td>
      <td>Baccio Bandinelli (Italian, 1493-1560)</td>
      <td>c. 1516-1520</td>
      <td>Italy</td>
      <td>This seated nude comes from a series of red chalk drawings that Baccio Bandinelli based loosely on Michelangelo's <em>ignudi</em> (nudes) on the Sistine Chapel ceiling (1508–12). <em>Seated Male Nude</em> is a finished work of art: the forms have precise outlines and are filled in with painstaking hatching that renders the figure in careful relief. One of the most important Italian sculptors of the 1500s, Bandinelli considered drawing his primary strength. His drawings were prized both during and after his lifetime by his many admirers and students (he opened one of the first drawing academies of Renaissance Italy). As he said himself, "All my concentration was fixed on drawing: in the judgement of Michelangelo, of our Princes, and other notables, it is above all in that activity that I have prevailed."</td>
      <td>Bandinelli's drawing of a moody nude combines his knowledge of ancient sculpture and paintings by Michelangelo with observation from life.</td>
      <td>This seated nude comes from a series of red chalk drawings that Bandinelli based loosely on Michelangelo's<em> ignudi</em> (nudes) on the Sistine Chapel ceiling (1508–12). <em>Seated Male Nude</em> is a finished work of art: the forms have precise outlines and are filled in with painstaking hatching that renders the figure in careful relief. Bandinelli prized these drawings and encouraged his heirs to save them for posterity. One of the most important Italian sculptors of the 1500, Bandinelli created works that reflect the influence of ancient models as well as a dependence on <em>disegno</em> (drawing). Bandinelli's drawings were prized both during and after his lifetime by his many admirers and students (he opened one of the first drawing academies of Renaissance Italy). As he said himself, "All my concentration was fixed on drawing: in the judgement of Michelangelo, of our Princes, and other notables, it is above all in that activity that I have prevailed."</td>
      <td>https://clevelandart.org/art/1998.6</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B2BC5BD15701CB07D3F06FE5B7163CDC2BFFE1E3F866DE38FEE56D65DB9951A9&s=21&se=1755401958&v=3&f=1998.6_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>11</th>
      <td>Nude</td>
      <td>David Alfaro Siqueiros (Mexican, 1896-1974)</td>
      <td>None</td>
      <td>Mexico</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1944.421</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5183C4D25C7FECF7FF4F166B05B6528DCD1347696A745CD4BABF8CDB0B45EF2D&s=21&se=1031267696&v=&f=1944.421_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>12</th>
      <td>Nude</td>
      <td>Alfeo Faggi (American, 1885-1966)</td>
      <td>1953</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.207</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA773164218693BAEBE51A9ECE691B51E4337D9E40F9342E6EAEA5D6C316&s=21&se=1391664542&v=&f=1956.207_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>13</th>
      <td>Nude</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1958.389</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E5DB3F754135455F4781116CF6C32CDA6A61CD42DC943CD03&s=21&se=1674154316&v=3&f=1958.389_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>14</th>
      <td>Nude</td>
      <td>James Roy Hopkins (American, 1877-1969)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.217</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C7480607DEA7BEED927AF927040FAF3BE61EB65363C59AC4158F97B3&s=21&se=1049999129&v=3&f=1923.217_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>15</th>
      <td>Seated Nude Woman</td>
      <td>Henri Matisse (French, 1869-1954)</td>
      <td>1924</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.543</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2FDEA15231BA420783211AEE730A54AE3F06308B4E8F7A9E0D03A79BAEE1CB0D&s=21&se=1049999129&v=&f=1926.543_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>16</th>
      <td>Nude</td>
      <td>Yasuo Kuniyoshi (American, 1889-1953)</td>
      <td>1922</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1971.184</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB119DB1FA8FF0F417CD77BCC5A3299F7E6C74F3EE23D17E05&s=21&se=1065507836&v=10&f=1971.184_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>17</th>
      <td>Nude</td>
      <td>Charles Salerno (American, 1916-1999)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1957.395</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB47A696F3CFB34AD32406FA530FA47104CC0463D29B6D5B49&s=21&se=1391664542&v=&f=1957.395_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>18</th>
      <td>Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.123</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C1E108216A5FC5C0CBEE799E775EB44E439640F6E215FEBD2&s=21&se=83547869&v=&f=1995.123_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>19</th>
      <td>Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.124</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233CB9C10FB0E03D4A9CC4D5EC53BA4E1DA0A08405FFB6AE2555&s=21&se=83547869&v=&f=1995.124_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>20</th>
      <td>Seated Nude</td>
      <td>Jean Louis Forain (French, 1852-1931)</td>
      <td>fourth quarter 1800s or first third 1900s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.274</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E8CCDDD76C98ED395E367A5803CF57D8FFC5D091376DBB995&s=21&se=1049999129&v=&f=1926.274_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>21</th>
      <td>Study for the Nude Youth over the Prophet Daniel (recto)</td>
      <td>Michelangelo Buonarroti (Italian, 1475-1564)</td>
      <td>1510-1511</td>
      <td>Italy</td>
      <td>None</td>
      <td>The drawing by Michelangelo made as preparatory for the Sistine Ceiling fresco is one of just four drawings by the master in an American museum.</td>
      <td>Universally considered one of the greatest artists of the Italian Renaissance, Michelangelo devoted four years to painting the vast ceiling fresco in the Sistine Chapel. This preparatory study portrays one of the 20 athletic male nudes, known as ignudi, who serve as supporting figures at each corner of the Old Testament scenes painted down the center of the ceiling. Michelangelo worked out the positioning of the ignudi in red chalk drawings before beginning to paint each section of wet plaster. The energy and monumentality of the figure in red chalk, whose body extends beyond the sheet, suggests the heroic athleticism of Michelangelo’s sculpture.</td>
      <td>https://clevelandart.org/art/1940.465.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CDE735E74C0ACE20FB1D913A8D2588BC63F53A0BD246CC89EA10BC895E52E7FB&s=21&se=1031267696&v=1&f=1940.465.a_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>22</th>
      <td>Nude</td>
      <td>Childe Hassam (American, 1859-1935)</td>
      <td>1918</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1940.120</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C74806078A5B8D23EBD96C8ADB716C7D247AE95478FC51EB569E0F1E&s=21&se=1031267696&v=3&f=1940.120_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>23</th>
      <td>Nude</td>
      <td>Adeline Wilkens David (American, 1911-)</td>
      <td>1949</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1949.71</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A1D29766D28121A5363A121F3E2070AE2B12A60EB41B23575E86D83638270809&s=21&se=1031267696&v=3&f=1949.71_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>24</th>
      <td>Nude</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1951.481</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5EEE5497B61866A22A94D2577DC914654A2583B7B3857764A6&s=21&se=1031267696&v=3&f=1951.481_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>25</th>
      <td>Nude Walking Like an Egyptian</td>
      <td>Karl F. Struss (American, 1886-1981)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Ancient Egyptians depicted the human figure following conventions that, to contemporary eyes, appear to view the same body from different perspectives. This and other Egyptian styles and motifs inspired artists, architects, designers, and even dancers in the 19th and early 20th century, as evidenced in Struss’s photograph. This figural study belongs to <em>The Female Figure</em>, a portfolio of nude studies that Struss published for artists to use. He employed three painters to consult on the poses, which were executed by models and dancers.</td>
      <td>https://clevelandart.org/art/2012.316</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=242775E0B50F25E80AABAE67FD160DD9369E552755341EF2E84AE13CE4E33C7C&s=21&se=1689619227&v=&f=2012.316_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>26</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1966</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.414</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1F448EBE0D6107CF57E4F48D619C8CA7D9C725535EB534AB5E4A9D5DEDC56F22&s=21&se=92257091&v=4&f=1985.414_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>27</th>
      <td>Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1940</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2004.147</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=976013C7DEFC9D5636DC603FF3A146AF61D4B64F175BB1F5B10615AB100115A2&s=21&se=1755401958&v=&f=2004.147_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>28</th>
      <td>Study of a Male Nude</td>
      <td>Pierre-Paul Prud'hon (French, 1758-1823)</td>
      <td>c. 1810</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.318.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BBD415F6B19E766DD5AC04504502B64FBDDA45DB3423FB10D&s=21&se=1674154316&v=&f=1961.318.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>29</th>
      <td>Study of a Reclining Nude</td>
      <td>Isidore Pils (French, 1813/15-1875)</td>
      <td>c. 1841</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>The languorous, sensuous pose of this woman is strongly reminiscent of Jean Auguste Dominique Ingres's popular paintings of odalisques, female slaves, and concubines in Turkish harems. Much of this canvas has been left thinly painted or entirely blank, suggesting that it was a figural study rather than a finished work of art. The French Academy in the 1800s viewed the depiction of the nude as the ultimate measure of an artist's skill. Because models changed poses frequently, students had to work quickly and without embellishment. Here the artist completed only those areas needed to emphasize the contours of the model's body.</td>
      <td>https://clevelandart.org/art/1939.63</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=51D1F5AF50C23DA5E2E6BE31D72EA3B16C9AD16B5B840EC381C1BD9D5E589224&s=21&se=1031267696&v=&f=1939.63_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>30</th>
      <td>Nude Study</td>
      <td>George Kolbe (German, 1877-1947)</td>
      <td>first quarter 1900s</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1928.202</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BC81A92BC8116C71EB60177AA86919AADFA276EED2E57572A&s=21&se=1049999129&v=&f=1928.202_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>31</th>
      <td>Reclining Nude</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>None</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1925.1272</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72565F72D1BC619106E97BC24A299AAB64A07F8FECF7085C69&s=21&se=1049999129&v=4&f=1925.1272_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>32</th>
      <td>Diagonal Nude</td>
      <td>Emil Nolde (German, 1876-1956)</td>
      <td>1908</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>This depiction of a nude is unusual; seen from above, her body bisects the plate diagonally in a bold, abstract way that is counter to the conventional ideal nude displayed frontally, lying on her side. Instead of using the traditional copper plate, Nolde used iron, which enhanced the corrosive appearance of the green color. He also unevenly laid the etching ground on the plate, resulting in the irregular tone in the background. The grainy, streaky marks around the woman’s body were drawn with a blunt point rather than the more typical sharp etching needle. His technique embraced the unpredictability and spontaneity of the etching process.</td>
      <td>https://clevelandart.org/art/1974.44</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=92B37231D833A780BD53BA4F77C2DB5E9402EBF49A85FB06F721F8B10AC83DA3&s=21&se=1065507836&v=3&f=1974.44_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>33</th>
      <td>Study of a Nude Woman, Seated Looking to the Right</td>
      <td>Pierre-Paul Prud'hon (French, 1758-1823)</td>
      <td>c. 1810</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.318.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BD970547AC89821020C3F632F4461D38A277138A54AC9C0EF&s=21&se=1674154316&v=3&f=1961.318.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>34</th>
      <td>Untitled (Nude)</td>
      <td>Frantisek Drtikol (Czech, 1883-1961)</td>
      <td>c. 1927-29</td>
      <td>Czechoslovakia</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.228</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E879269A3676895AEE5A349E9B47561352AFE72BCD9F651058E4EF0F2BCF266&s=21&se=525383189&v=4&f=1992.228_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>35</th>
      <td>Untitled (Nude)</td>
      <td>Alexis-Louis-Charles-Arthur Gouin (French, 1855)</td>
      <td>1851-1852</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>A master of commercial portrait photography, Gouin specialized in hand-painted stereoscopic daguerreotypes. Derived from the inventions of Sir Charles Wheatstone (1802-1875) in 1832 and Sir David Brewster (1781-1868) in 1849, the stereoscopic technique produced two almost identical photographic images. When seen simultaneously in a viewing instrument called a stereoscope, the resulting effect was an astonishing illusion of three-dimensional space.\r\n\r\nThis rare example comes from an important group of nude studies Gouin created in the early 1850s. To create the image, Gouin carefully posed a favorite model--Delphine Herbé, a florist--in his third-floor studio where the bright, natural light would define her body. Drapery was used to relieve the monotony of the background and heighten the three-dimensionality of the model's form and echo its contours. Gouin's training as a painter of miniatures is evident in the delicacy of the hand-coloring and in the subtle, naturalistic application of pigment over the polished surface of the plate.\r\n</td>
      <td>https://clevelandart.org/art/1999.8</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DEE1FE91B873FA061962166169045956576C12A243C75EB96&s=21&se=1755401958&v=4&f=1999.8_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>36</th>
      <td>Nude Woman Standing</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.544</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E5324C2CC0A7F194388171E027199EE885C30331B42692458&s=21&se=1049999129&v=3&f=1926.544_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>37</th>
      <td>Reclining Nude</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1938</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1963.151</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BFD5A59DC96353CFE2C1597CE3E93161629383A46D7A9DF80&s=21&se=1674154316&v=5&f=1963.151_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>38</th>
      <td>Male Nude</td>
      <td>August Macke (German, 1887-1914)</td>
      <td>1912</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.314</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BC23164687437F421179E585895A6618D686E8F1748EF2222&s=21&se=1065507836&v=&f=1972.314_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>39</th>
      <td>Male Nude</td>
      <td>Guglielmo Marconi (French, 1842-aft.1885)</td>
      <td>c. 1870-1875</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>An Italian photographer active in Paris from the mid-1960s through the mid-1870s, Guglielmo Marconi specialized in academic nude studies for painters and sculptors.  Using child, female, and male models, he photographed an entire repertory of poses traditionally employed by academic studios. Illustrating the developing close association between photographers and painters, Marconi's image is related to another nude study in the collection, Julien Vallou de Villeneuve's Study after Nature (Etude d'après nature).</td>
      <td>https://clevelandart.org/art/1996.357</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57D109E894230FB7EACBFD24E2848944B766B02FC3AEC8C717&s=21&se=83547869&v=3&f=1996.357_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>40</th>
      <td>Kneeling Nude</td>
      <td>Arthur B. Davies (American, 1862-1928)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1931.236</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E5AAF2320DE9C8FD9854E652C984A352B3306D57472487462&s=21&se=1049999129&v=&f=1931.236_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>41</th>
      <td>Nude at the Shore</td>
      <td>Erich Heckel (German, 1883-1970)</td>
      <td>1913</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>The nude woman in this print is outlined in blue watercolor, hinting at the presence of water. In the early years of their association, Erich Heckel and his fellow Die Brücke artists often visited the wooded Moritzburg ponds outside Dresden, a locale known for its bohemian lifestyle that included sunbathing in the nude. Heckel made this image on one of those trips, when he likely produced the sketch and carved the woodblock outdoors.</td>
      <td>https://clevelandart.org/art/1986.16</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=92B37231D833A7806E70E2DEA524E41384997BF36FCA3F1AB46BA0DD8A0B87B0&s=21&se=166975769&v=&f=1986.16_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>42</th>
      <td>Reclining Male Nude</td>
      <td>Gaetano Gandolfi (Italian, 1734-1802)</td>
      <td>second half 18th century</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1962.19</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F93CB46F71431BEE2C37EEDD9FD1EF51E9E1AB0D054B03101AA44433235DC4B9&s=21&se=1674154316&v=&f=1962.19_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>43</th>
      <td>Nude Woman</td>
      <td>Alphonse Legros (French, 1837-1911)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.484</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932EFDA563C861C848FDAA6A7EB9DF38799BB1CB17810342AB41&s=21&se=1049999129&v=&f=1926.484_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>44</th>
      <td>Reclining Nude</td>
      <td>Lennart Anderson (American, 1928-2015)</td>
      <td>1962</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1980.126</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB8AB57EA7EF77F1ED7F5487D8C08F1890455655AD6F42532A&s=21&se=1329612074&v=4&f=1980.126_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>45</th>
      <td>Nude Man Standing, with Left Hand Raised</td>
      <td>Edgar Degas (French, 1834-1917)</td>
      <td>c. 1900</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.316</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BF50A13B21965C8A4641A460F173DC273313D65B73334E2A7&s=21&se=1031267696&v=&f=1941.316_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>46</th>
      <td>Seated Nude</td>
      <td>Tsugouharu Foujita (French, 1886-1968)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.268</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A9D32762E79BC6862A2FF401DE8FAC1ADBC2DC0EF81A776844A1E7DAEDC473DB&s=21&se=1031267696&v=3&f=1941.268_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>47</th>
      <td>Nude Woman</td>
      <td>John Bernard Flannagan (American, 1895-1942)</td>
      <td>1927</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1958.70</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDEAB5060FF48E76A40E13B9DD05A26184FE0883F524A746CB1&s=21&se=1674154316&v=&f=1958.70_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>48</th>
      <td>Seated Nude</td>
      <td>William Dean Fausett (American, 1913-1998)</td>
      <td>1933</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1951.55</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186ECFF091D957AD9196278BD217F65AC4300D41DD31130FF0C&s=21&se=1031267696&v=2&f=1951.55_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>49</th>
      <td>Male Nude</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>1891</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1986.131</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2A6A5B621F60D7F40F8C2EB8B8DA9C638809848CD3CD8D5CB7147DAFBFE815DF&s=21&se=951169631&v=&f=1986.131_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>50</th>
      <td>Reclining Female Nude</td>
      <td>Amedeo Modigliani (Italian, 1884-1920)</td>
      <td>1914</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1950.456</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F93CB46F71431BEEA5DCAD6AB710685EBE1F3424093134E27AFF56E3A9AEFEF6&s=21&se=1031267696&v=2&f=1950.456_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>51</th>
      <td>Nude Woman</td>
      <td>Eugene Edward Speicher (American, 1883-1962)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1943.310</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186AF517ACBEF516D93BE62C76089470D592D020822D3DB102D&s=21&se=1031267696&v=2&f=1943.310_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>52</th>
      <td>Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.32</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE112583AF604E22ED060223C5C24A45442E680B124B6712D8DEAA6D&s=21&se=1049999129&v=&f=1926.32_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>53</th>
      <td>Reclining Nude</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1958.390</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E8B454D06216391EAA8D8BE9F9331D6D8C5E46A675A370A8C&s=21&se=1674154316&v=4&f=1958.390_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>54</th>
      <td>Nude, Front</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.112</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C4C0F86E6D980AE5AB954972898CC6DD133237CFA68F94815&s=21&se=1329612074&v=&f=1978.112_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>55</th>
      <td>Standing Female Nude</td>
      <td>John Sloan (American, 1871-1951)</td>
      <td>1920s</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>John Sloan achieved fame as a member of the Ashcan School, painting scenes of urban life. However, he devoted much of his later career to representations of nudes.</td>
      <td>https://clevelandart.org/art/1948.154</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB22E036BBDA99A2C98A84C3B7A833E1BFEECDA82CB01B93AB&s=21&se=1031267696&v=2&f=1948.154_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>56</th>
      <td>Nude Study</td>
      <td>William Orpen (Irish, 1878-1931)</td>
      <td>1900</td>
      <td>Ireland</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1925.439</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4B32F978B28D5E5222A7BCB35B501749D270710555684C2BA72F0A3260BBD064&s=21&se=1049999129&v=2&f=1925.439_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>57</th>
      <td>Female Nude</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>1891</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1986.130</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2A6A5B621F60D7F4771D0790BB5C762CE5EDEF328470F50A62F75286DC21CF30&s=21&se=166975769&v=&f=1986.130_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>58</th>
      <td>Stooping Nude</td>
      <td>Albert Edward Sterner (American, 1863-1946)</td>
      <td>1915</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1922.387</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA77316421862E7E03E90481D6BEE86D3848A9995014540899840FB212E5&s=21&se=1049999129&v=&f=1922.387_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>59</th>
      <td>Seated Nude Study</td>
      <td>Alexander Archipenko (American, 1887-1964)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1932.418</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E8A3672D90C6A0CBE898E8244A5FB87C55A458A52528ED079&s=21&se=1049999129&v=2&f=1932.418_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>60</th>
      <td>Female Nude</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1924.614</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C520EE3F0B1C2588106C275B5B0BC4512E805F3C1C04AC4B5&s=21&se=1049999129&v=2&f=1924.614_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>61</th>
      <td>Reclining Nude</td>
      <td>Emil Orlik (Czech, 1870-1932)</td>
      <td>None</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1924.256</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CFB28CA2706BD73CE8FC5CD21781DBD118F1BF94D6F222F41D25C356437E9052&s=21&se=1049999129&v=7&f=1924.256_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>62</th>
      <td>Nude Study</td>
      <td>Dan Hodermarsky (American, 1924-)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1947.334</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C74806079155393EFFFE401FDE7D0117B6D81042AABDD1CEACED3940&s=21&se=1031267696&v=&f=1947.334_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>63</th>
      <td>Nude Reading</td>
      <td>Anne Goldthwaite (American, 1869-1944)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1949.395</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9FD37BAFC27E9FE170E4336E9C7EDE1607009B4B80ED7E81DA3E6B7C96C46EC1&s=21&se=1031267696&v=3&f=1949.395_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>64</th>
      <td>Nude with Bracelet</td>
      <td>Henri Matisse (French, 1869-1954)</td>
      <td>c. 1940</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Nude with Bracelet exemplifies the basic advantage of linoleum cutting: the soft material makes it easy to produce a continuous, fluid line. The image was cut from the block, the surface was inked in black, and it was printed on white paper.</td>
      <td>https://clevelandart.org/art/1981.41</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E3471A387F779686D54BDCF1643828DE7C3C66CA9C8585681&s=21&se=1329612074&v=4&f=1981.41_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>65</th>
      <td>Nude, Back</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.111</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C2A3F4F9003FBBB9860C670A388874751AF6EF5E7AE4CA585&s=21&se=1329612074&v=&f=1978.111_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>66</th>
      <td>Reclining Nude</td>
      <td>Henri Matisse (French, 1869-1954)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1959.27</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5EBB0AC45E1AC5C31BED49C4EE2FC1B051E7EC01147092AE67&s=21&se=1674154316&v=4&f=1959.27_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>67</th>
      <td>Nude, London</td>
      <td>Bill Brandt (British, 1904-1983)</td>
      <td>1952</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2011.171</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2A6A5B621F60D7F4D18DC586448AF4C4B9C10F86BD7529E7BE1168C689D7CC7E&s=21&se=1502064348&v=&f=2011.171_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>68</th>
      <td>Nude, Phoenix, Arizona</td>
      <td>Lee Friedlander (American, b. 1934)</td>
      <td>1978</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1989.427</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762B8328CC1CD5D43D461FA8A00294277C9C3AFB2C0157B103B6&s=21&se=447131495&v=3&f=1989.427_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>69</th>
      <td>Reclining Nude</td>
      <td>Emil Fuchs (American, 1866-1929)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.184</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9FD37BAFC27E9FE1E148AB14F9B8F8E3F0FD943EFBABE58E8CCFCF958DBA1C6B&s=21&se=1049999129&v=4&f=1926.184_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>70</th>
      <td>Nude Figure</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>c. 1927</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1927.177</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C23C2E95D44874B849B5C6A291481C86BA156779825BB9180&s=21&se=1049999129&v=2&f=1927.177_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>71</th>
      <td>Nude Women</td>
      <td>Camille Pissarro (French, 1830-1903)</td>
      <td>c. 1896</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1973.124</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=96B479B259297BE98A67C966158D110F934B0BB169403EB8FDDEB15568561287&s=21&se=1065507836&v=5&f=1973.124_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>72</th>
      <td>Nude Fish</td>
      <td>Hugh Seaver</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1924.908</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A4E391AF09FB11301989590E16FDCDFCB6D2E3BC110187497A&s=21&se=1049999129&v=4&f=1924.908_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>73</th>
      <td>Reclining Nude</td>
      <td>William Strang (British, 1859-1921)</td>
      <td>None</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1935.26</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4B32F978B28D5E52EF88CDE5650CE533059F6F0CCDEFBE80BC005E5754BF2FAD&s=21&se=1049999129&v=2&f=1935.26_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>74</th>
      <td>Nude Woman</td>
      <td>Boardman Robinson (American, 1876-1952)</td>
      <td>c. 1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1986.125</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932EFE8A5AFCAF2D86D641E9BC1EADEDAE11C918A6EE0FD76AAD&s=21&se=166975769&v=2&f=1986.125_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>75</th>
      <td>Female Nude</td>
      <td>Dorothy Wilding (British, 1893-1976)</td>
      <td>None</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.1085</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=40675183989CEBD8F2875CDBCE742ABE17058BD905860BD1C9467023D5A51EF7&s=21&se=1065507836&v=3&f=1972.1085_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>76</th>
      <td>Study of a Male Nude</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>c. 1907</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1950.454</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932EF552E013C2CAE3E1095FFDD79D99CA813842D0E416BCF544&s=21&se=1031267696&v=&f=1950.454_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>77</th>
      <td>Nude Women</td>
      <td>Camille Pissarro (French, 1830-1903)</td>
      <td>c. 1896</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1973.125</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=96B479B259297BE9206F3A7B9F5A770CA3874D19BE6B84F9A04145B86489127F&s=21&se=1065507836&v=5&f=1973.125_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>78</th>
      <td>Seated Nude</td>
      <td>William Etty (British, 1787-1849)</td>
      <td>None</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.33</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4B32F978B28D5E521B4260D7A2F353670F4DF90D90E7F930A3799A5587CCE655&s=21&se=1329612074&v=2&f=1978.33_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>79</th>
      <td>Standing Nude</td>
      <td>Alexander Archipenko (American, 1887-1964)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1979.99</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932EB12D79AA62880A6DB74BC8E40CE74E7C85B4150B49E68974&s=21&se=1329612074&v=&f=1979.99_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>80</th>
      <td>Nude in Light</td>
      <td>Franz Roh (German, 1890-1965)</td>
      <td>1925</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>An example of montage, this image was produced by stacking several negatives together and printing through the sandwich. Roh experimented freely with different techniques to obtain the widest range of psychological expression. He described the technique of photomontage: "from simple fragments of reality, a more complex unit is piled up." This process, he believed, was "based on a deep need of human imagination."</td>
      <td>https://clevelandart.org/art/2007.111</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CFF185746D099B0B5359BFC02EB5C1F1C386BF9CD2D0B97E5C&s=21&se=653649514&v=&f=2007.111_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>81</th>
      <td>Nude Sleeping</td>
      <td>Leon Kroll</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1945.254</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E705461AD9BEA33C11DBCFDB30E74391D0ABED773DEC3E196&s=21&se=1031267696&v=&f=1945.254_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>82</th>
      <td>Seated Nude</td>
      <td>David Alfaro Siqueiros (Mexican, 1896-1974)</td>
      <td>None</td>
      <td>Mexico</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.299</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5183C4D25C7FECF73D87B2C68906F31F817F98814FF08F6AD65EEC263A7C6654&s=21&se=995010993&v=&f=1956.299_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>83</th>
      <td>Nude Woman in a Landscape</td>
      <td>André Derain (French, 1880-1954)</td>
      <td>first quarter 1900s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.545</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504B6F97A9F29CD4CB766F1003BF97786B9D11458AE6DCB552F0&s=21&se=1049999129&v=&f=1926.545_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>84</th>
      <td>Walking Nude</td>
      <td>Charles Frederick Ramus (American, 1902-1979)</td>
      <td>1930</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1930.151</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A4AE21507FDAEE7E634096BD1AD1476142C0B17D046DA8C9F0&s=21&se=1049999129&v=&f=1930.151_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>85</th>
      <td>Reclining Nude</td>
      <td>Othone Coubine (French, 1883-1969)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1965.59</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=33B458E6429A5D59E8A24E2F88CC25E1980F01463C6CB04242C1917A5C4624CA&s=21&se=1120217757&v=3&f=1965.59_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>86</th>
      <td>Reclining Nude</td>
      <td>André Lhote (French, 1885-1962)</td>
      <td>20th century</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2009.297</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=36B59A5C833AF0E322904D58E9242E8AD629BEAC79698D0AF9E5BE15E65FD91C&s=21&se=1285208684&v=3&f=2009.297_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>87</th>
      <td>Standing Male Nude</td>
      <td>Frederick William MacMonnies (American, 1863-1937)</td>
      <td>1885</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>The sculptor Frederick MacMonnies is chiefly known for his exuberant, sensual nudes, such as those decorating the huge fountain of 1893 for the World's Columbian Exposition in Chicago, which featured dozens of frolicsome, stark-naked female figures. Curiously, his drawings reveal a different side of his talent—a detailed, slightly somber realism, which brings out the unidealized individuality of his models.</td>
      <td>https://clevelandart.org/art/1939.545</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57FC8F75F5C32741C84C7F7811D659A0CEF4AF18EF7EC919E1&s=21&se=1031267696&v=&f=1939.545_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>88</th>
      <td>Study of a Nude Woman, Seated Looking to the Right (recto) Study of a Male Nude (verso)</td>
      <td>Pierre-Paul Prud'hon (French, 1758-1823)</td>
      <td>c. 1810</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1961.318</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=3C8119C3BF4A0485D5B2E2B60B18A29C28617B78DCF0919F0DA0EE7DCD397515&s=21&se=1674154316&v=&f=1961.318_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>89</th>
      <td>Seated Female Nude</td>
      <td>Suzanne Valadon (French, 1865-1938)</td>
      <td>first third 1900s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.658</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA773164218648EBB7C8FBFF99BC813B85060A1B02D85DFC4A3F3E8E2005&s=21&se=995010993&v=&f=1954.658_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>90</th>
      <td>Seated Female Nude (Self-Portrait?)</td>
      <td>Paula Modersohn-Becker (German, 1876-1907)</td>
      <td>c. 1899</td>
      <td>Germany</td>
      <td>Although Paula Modersohn-Becker died in 1907, just as the Expressionist groups in Dresden and Munich were forming, the themes of her work prefigure the movement. This likely self-portrait exhibits her desire to convey not the idealized appearance of the female body but rather its fundamental essence, stripped of all the world’s trappings. She distilled the human body into flattened forms—achieved by erasing and blending the charcoal—and abbreviated the delineation of the feet, hands, and face. The sitter’s piercing stare invites the viewer to move beyond the body as flesh and blood toward her emotional or spiritual state.</td>
      <td>Paula Modersohn-Becker's career was extremely brief but prolific before dying from complications of childbirth at age 31.</td>
      <td>Although Paula Modersohn-Becker died in 1907, just as the Expressionist groups in Dresden and Munich were forming, the themes of her work prefigure the movement. This likely self-portrait exhibits her desire to convey not the idealized appearance of the female body but rather its fundamental essence, stripped of all the world’s trappings. She distilled the human body into flattened forms—achieved by erasing and blending the charcoal—and abbreviated the delineation of the feet, hands, and face. The sitter’s piercing stare invites the viewer to move beyond the body as flesh and blood toward her emotional or spiritual state.</td>
      <td>https://clevelandart.org/art/1973.35</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF49A1628255F7D6C9B5D33585C4FF8D5A4B7DDA1F3BA2B3A2&s=21&se=1523204814&v=&f=1973.35_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>91</th>
      <td>Nude on Dahomey Stool</td>
      <td>Philip Pearlstein</td>
      <td>1976</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This print is related to the museum's 1976 painting Female Model on African Stool, on view in Gallery 240.</td>
      <td>https://clevelandart.org/art/1999.275</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=419134297A23A50B7B144DD0C5D7CC8B29DB5F5DE620508E4076D2697650E8C8&s=21&se=1755401958&v=1&f=%5Cd6736%5Cu2767361%5C1999.275_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>92</th>
      <td>Seated Nude</td>
      <td>Alfred J. Wands (American, 1904-1998)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.170</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186518BC26FD67425FC509558CEE0D256133A59CAF6DB52C7AC&s=21&se=1049999129&v=2&f=1926.170_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>93</th>
      <td>Nude Resting</td>
      <td>Albert Edward Sterner (American, 1863-1946)</td>
      <td>1930</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1948.431</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A4E2B0B46DF0F1FAD41B34E276382DAD9E1002E0F9A817D2BC&s=21&se=1031267696&v=3&f=1948.431_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>94</th>
      <td>Nude Sculpture</td>
      <td>R. B. Kitaj (American, 1932-2007)</td>
      <td>1975</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1984.103</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7DC9D9A3CD012D6ACF1DFC6595E76E2BDEE13C3C7D63F7DFBB&s=21&se=933923230&v=1&f=%5Cd6253%5Cu2962539%5C1984.103_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>95</th>
      <td>Nude Study</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>1881-1944</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2010.293</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5D0F103AF8C233421BEACA61E4391821A1591D948312A9BC93CF0E2DBA228B99&s=21&se=1171617854&v=&f=2010.293_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>96</th>
      <td>Nude Kicking</td>
      <td>Karl F. Struss (American, 1886-1981)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2012.315</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=242775E0B50F25E81840B5F11F705F306A8E549B814FD01F6081BFED48C50BE8&s=21&se=1053970089&v=&f=2012.315_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>97</th>
      <td>Reclining Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.39</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE1125833EFB865F788EF5A12E8BF6FB8E79474E192C1857A226CD5E&s=21&se=1049999129&v=&f=1926.39_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>98</th>
      <td>Seated Female Nude</td>
      <td>Alexander Archipenko (American, 1887-1964)</td>
      <td>c. 1929</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1973.239</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F8BC7E8397D9E6645A5FE3D597462522A222C168B4883633AAAE9A0E05986A13&s=21&se=1065507836&v=&f=1973.239_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>99</th>
      <td>Nude Woman Seated on a Bed</td>
      <td>Suzanne Valadon (French, 1865-1938)</td>
      <td>first third 20th century</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1953.574</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2A6A5B621F60D7F4FD7FCFD6E5120BC4C3B87820D03525F821AC27DFD915AA3B&s=21&se=1031267696&v=&f=1953.574_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>100</th>
      <td>Nude Woman Standing</td>
      <td>Frederick William MacMonnies (American, 1863-1937)</td>
      <td>1885</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1939.546</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA77316421868290135B84A9A51430E160B868FFB308F1DFF99922F263C0&s=21&se=1031267696&v=&f=1939.546_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>101</th>
      <td>"Nude Foot" (San Francisco)</td>
      <td>Light Gallery</td>
      <td>1947</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1986.221.2</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=47099380FD68FF9822ABF3A897B90F44DBE8E166476A812555C3A573DFA0D874&s=21&se=508452570&v=2&f=1986.221.2_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>102</th>
      <td>Nude #1, 2, 3: Nude # 2</td>
      <td>Rizzoli</td>
      <td>1984</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2009.594</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EF640E54D64F5E9972917250779E01BEC129598FD8CA8F3DC426225975D0F3F2&s=21&se=1579770806&v=&f=2009.594_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>103</th>
      <td>Nude #1, 2, 3: Nude # 3</td>
      <td>Rizzoli</td>
      <td>1984</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2009.595</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EF640E54D64F5E99EA29100973F4F0803976F1A50623F60C4D4DD6F1311DA579&s=21&se=1579770806&v=&f=2009.595_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>104</th>
      <td>Reclining Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.34</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE112583A13B4D1FB375F5D842826FAEB757A810D95F341D5CA27193&s=21&se=1049999129&v=&f=1926.34_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>105</th>
      <td>Nude, No. 1</td>
      <td>Manuel G. Silberger (American, 1898-1968)</td>
      <td>1930</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1930.163</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A40E0DBDBE7E821653E9ADCFF700652994F17679572B6A5085&s=21&se=1049999129&v=3&f=1930.163_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>106</th>
      <td>Nude Model Reclining</td>
      <td>James McNeill Whistler (American, 1834-1903)</td>
      <td>1893</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1924.200</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7D20843ADEFFEE9F95859054292100C3FDB9E357CD5EB1F014&s=21&se=1049999129&v=4&f=1924.200_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>107</th>
      <td>Nude Female Dancers from a Tunic</td>
      <td>NaN</td>
      <td>700s</td>
      <td>Egypt or Syria</td>
      <td>None</td>
      <td>None</td>
      <td>Two rare nude female dancers with anklets and long scarves draped around their shoulders hold branches and pomegranates in their hands. This silk square originally decorated a tunic.</td>
      <td>https://clevelandart.org/art/1953.332</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD191B442CC39CBF088DDA6D5F009D98F54C1E50A17F97762DF&s=21&se=1031267696&v=&f=1953.332_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>108</th>
      <td>Nude, Vasterival, Normandy</td>
      <td>Bill Brandt (British, 1904-1983)</td>
      <td>1954</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>The distorted, dream-like reality here is in part due to the framing, the wide angle, and deep focus of the camera used by Brandt, which was originally designed to photograph crime scenes. Taken two decades after his photograph of the Barcelona beggar in the exhibition’s City section, this image belongs to a series of female nudes shot on the beaches of the English Channel.</td>
      <td>https://clevelandart.org/art/2007.36</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF3763C6FBDD77C1D117A1FDB361D36C7B6BBA40C5A1252802&s=21&se=1828349793&v=&f=2007.36_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>109</th>
      <td>Nude and Cock</td>
      <td>Jacques Lipchitz</td>
      <td>1945</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1994.299</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=40675183989CEBD82227A2253F6B668D7BECC2A30AF752C91C396571FE2E8809&s=21&se=83547869&v=4&f=1994.299_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>110</th>
      <td>Semi-nude Woman, Seated</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.33</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE1125837F0316BF6D656EBC43DAAD606F0FD2F1272E71CC53349CA0&s=21&se=1049999129&v=&f=1926.33_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>111</th>
      <td>Nude Foot (San Francisco)</td>
      <td>Minor White (American, 1908-1976)</td>
      <td>1947</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1993.18</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=47099380FD68FF98D140289855E0597C297B7546F4619BE116EF8FD894CCF6B9&s=21&se=1725088047&v=4&f=1993.18_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>112</th>
      <td>Reclining Male Nude (verso)</td>
      <td>Edouard Vuillard (French, 1868-1940)</td>
      <td>c. 1898</td>
      <td>France</td>
      <td>During the early 1890s, Édouard Vuillard repeatedly represented his mother and older sister, Marie, in domestic settings. This drawing relates to a pastel and painting, all of which show Marie standing before a mirror, raising her arms as if to adjust her hairstyle. A charcoal study of a nude male appears on the verso, suggesting that the artist may have reused the sheet.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.77.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=81064D36B0DB7492D52E9C59675425104D0835C8A666FFA4B3649FEAA039AE3A&s=21&se=402160102&v=1&f=2018.77.b_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>113</th>
      <td>Standing Female Nude</td>
      <td>Otto H. Bacher (American, 1856-1909)</td>
      <td>probably 1878-79</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Otto Bacher was the first Cleveland artist to achieve a national reputation. He probably made this drawing while studying at the Royal Academy in Munich.</td>
      <td>https://clevelandart.org/art/1983.243</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB6908C8153743FDF71E18A3BBACFD96F3BE41F3ED18E68825&s=21&se=933923230&v=&f=1983.243_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>114</th>
      <td>Reclining Nude (facing right)</td>
      <td>Max Weber (American, 1881-1961)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.273</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7D098A44D47A046F965DDACC63D9108DC851CA982FFA56FE9D&s=21&se=1031267696&v=4&f=1941.273_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>115</th>
      <td>Nude Man and Woman</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1925.1034</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5EC3447BBF597851C593ACA0D86CFF894C53C3A928D96FC97A&s=21&se=1049999129&v=4&f=1925.1034_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>116</th>
      <td>Nude on Draped Couch</td>
      <td>John Sloan (American, 1871-1951)</td>
      <td>1931</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1949.87</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0E69E73DAC21794DAADE96A4846D4994C6DF3D9876E5C7D681E013D76C78EFB4&s=21&se=1031267696&v=3&f=1949.87_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>117</th>
      <td>Reclining Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.38</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE11258378466B28F6BFB69AE5AB60CEB8CA09BD7BD50740D776A6C4&s=21&se=1049999129&v=&f=1926.38_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>118</th>
      <td>Reclining Nude (facing left)</td>
      <td>Max Weber (American, 1881-1961)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.274</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7DE646CA04C4392C067E34F31F2232575EF27AF46C7FA9DBD5&s=21&se=1031267696&v=4&f=1941.274_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>119</th>
      <td>Study - Female Nude Standing</td>
      <td>Abel George Warshawsky (American, 1883-1962)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.165</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8CD75251BA0803429AE3B5D61E45BF27E315F21C97D80F63D6&s=21&se=1049999129&v=2&f=1923.165_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>120</th>
      <td>Camera Work: Nude</td>
      <td>W.W. Renwick (American, 1864-1933)</td>
      <td>1907</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.20.j</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9C187DB24CE47569F171DFB161B18170BA52348C8E85B395479F5928C86B9221&s=21&se=83547869&v=2&f=1995.199.20.j_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>121</th>
      <td>Transsexual Nude, Beijing</td>
      <td>Zheng Liu (Chinese, b. 1969)</td>
      <td>1998</td>
      <td>China</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2012.110</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D38E242A5205E8E9C5AD563E3F46E1B16A0E5AA409D51799E305194EFAD3AFEB&s=21&se=1502064348&v=&f=2012.110_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>122</th>
      <td>Seated Male Nude</td>
      <td>Oscar Miestchaninoff (French, 1886-1956)</td>
      <td>c. 1939</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1957.394</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E6F1A006554BB801AC7407A9942FCE44BFAF9ADEFCA599DF7&s=21&se=1391664542&v=&f=1957.394_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>123</th>
      <td>Standing Nude (verso)</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>1914</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.830.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C5F85A038E7E047DB9E20DCCCF032E8D07BE446A30A196951&s=21&se=1031267696&v=&f=1954.830.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>124</th>
      <td>Sea Nude, Camargue</td>
      <td>Lucien Clergue (French, 1934-)</td>
      <td>1962</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1996.415</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FF4C35FADDCC7EC409D8E68A9B0AE9C7D38CECBB78EC0F7C2E&s=21&se=686303180&v=4&f=1996.415_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>125</th>
      <td>Nude Boy on Horseback</td>
      <td>Renée Sintenis (German, 1888-1965)</td>
      <td>None</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.136</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72903886D621D190AB10F5E4D552304D7437073623FB5C21BC&s=21&se=1031267696&v=3&f=1941.136_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>126</th>
      <td>Nude Woman Standing</td>
      <td>John Bradley Storrs (American, 1885-1956)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1953.568</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A4448926889AB04073380C85D181C3D2E8909068E9076F6366&s=21&se=1031267696&v=3&f=1953.568_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>127</th>
      <td>Reclining Nude with Slave</td>
      <td>Marcel Meys (French)</td>
      <td>c.1905-1910</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2009.475</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EF640E54D64F5E9928C1B181210CA956D4FA5B3B57005E5B6B8E32FD3E9EF3F0&s=21&se=1285208684&v=&f=2009.475_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>128</th>
      <td>Nude Man Playing a Violin</td>
      <td>Jose Garcia Hidalgo (Spanish, 1646-1717)</td>
      <td>1693</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1991.240</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7255CAFE35B1A03C8C83864E685A9F5CA0FF81A9CCDC990010&s=21&se=525383189&v=4&f=1991.240_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>129</th>
      <td>Academic Nude, Reclining on a Sofa</td>
      <td>Auguste Belloc (French, 1801-c. 1868)</td>
      <td>c. 1855</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Before becoming a photographer and writer on photographic theory, processes, and techniques, Belloc produced watercolors and miniatures. By 1851, he had established a commercial studio in Paris, specializing in portraiture and nude studies. In this example, he carefully positioned his model with favorite props, such as a chaise longue and copious amounts of rich drapery. Natural illumination in the skylit studio clearly defines her sensuous body with bright highlights and rich shadows. The luxurious flesh tones and the suggestive pose add to the picture's eroticism. Shown in exhibitions and purchased by wealthy voyeurs, Belloc's images of nudes were also sought by painters wanting to avoid the expense of a live model.</td>
      <td>https://clevelandart.org/art/1998.55</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=6DC904522298CEDB1A94C556A39D35B6210DA57AF1AEB737C93AE262AE2AEB47&s=21&se=1755401958&v=4&f=1998.55_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>130</th>
      <td>Seated Male Nude</td>
      <td>Claude Gillot (French, 1673-1722)</td>
      <td>1693-1695</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2008.353</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B69E2CB8AE8587229DAA1F8E474EC3D779D6F27687CFEB4BC40731FEE5705B28&s=21&se=1285208684&v=2&f=2008.353_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>131</th>
      <td>Reclining Female Nude (recto)</td>
      <td>Anonymous</td>
      <td>19th century</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1989.242.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E0F8A78917125819D6726BBA7E76AB382398F8731C99ACC94&s=21&se=612257026&v=2&f=1989.242.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>132</th>
      <td>Nude Man and Woman Outside</td>
      <td>Lynd Ward (American, 1905-1985)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1988.136</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7D616C6ADFED3CDA29FF63BF808D431272B125E86440E5CEFF&s=21&se=1065507836&v=5&f=1988.136_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>133</th>
      <td>Couple in Room, Nude Man with Woman</td>
      <td>Ernst Ludwig Kirchner (German, 1880-1938)</td>
      <td>1915-1916</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>The man in this image resembles one of Ernst Ludwig Kirchner’s close friends, Hugo Biallowons, who was killed in World War I shortly after this print was made. Kirchner depicted the man in a cramped space next to a nude woman who reaches for him without making eye contact. The yellow paper suggests artificial light.  Kirchner owned one large lithographic stone, from which he made one lithograph after another simply by sanding down the stone, and then starting again with a new composition. This resulted in very small editions. By spreading water mixed with turpentine over the surface of the stone to loosen the lithographic crayon, he created unexpected and spontaneous results, seen here in the grainy and broken lines.</td>
      <td>https://clevelandart.org/art/1991.24</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09BE0BA75D28988E6205CBA77AA32EE86B9270E76B30E049914AD050DBB72184&s=21&se=1781306598&v=&f=1991.24_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>134</th>
      <td>Fifteen Nude Children Dancing</td>
      <td>Heinrich Aldegrever (German, 1502-1555/61)</td>
      <td>1535</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1922.127</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=BDFDD69D477608B1E60EF0BC5F1AFEB7041625A4BAEB3FB3F46BBD9F08EB009B&s=21&se=1049999129&v=6&f=1922.127_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>135</th>
      <td>Study - Female Nude Seated</td>
      <td>Abel George Warshawsky (American, 1883-1962)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.168</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C74C378CE55EAF3D85933740F0B85AD4614E6F6D1B35A37B7&s=21&se=1049999129&v=2&f=1923.168_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>136</th>
      <td>Morning, Nude Sketch</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1921</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.602</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E10902A90CCC4D196A08B0962CA96B5DB80CF7B3296CBD1671B2&s=21&se=1049999129&v=3&f=1936.602_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>137</th>
      <td>Nude Woman Seated</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1916</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.652</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E10906317E8B67C3286EADACE308B801708BC14541542939989A&s=21&se=1049999129&v=&f=1936.652_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>138</th>
      <td>Two Nude Women</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.211</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA724883AEA25557EF22BC0C5050BAEC519473D8B7A4C8B3D6D2&s=21&se=1065507836&v=5&f=1972.211_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>139</th>
      <td>Nude #1, 2, 3: Nude # 1</td>
      <td>Rizzoli</td>
      <td>1984</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2009.593</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EF640E54D64F5E99D6E4C87E6EAEDD66A5796DC793F5AD67B7AFE1D9FEAAFA34&s=21&se=1285208684&v=&f=2009.593_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>140</th>
      <td>Reclining Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.35</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE112583083849C955DD0033874EAFC8964AE0C5C6990AD9DB5256FE&s=21&se=1049999129&v=&f=1926.35_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>141</th>
      <td>Reclining Nude Woman</td>
      <td>Ananda K. Coomaraswamy (Ceylonese, 1877-1947)</td>
      <td>early 20th century</td>
      <td>Ceylon</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.36</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=12B106AAEE112583931E23056AAAF6337F07AA6FB8268E594F1C0BBC1DDCF68D&s=21&se=1049999129&v=&f=1926.36_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>142</th>
      <td>Torso of a Nude Woman</td>
      <td>Jane Poupelet (French, 1878-1932)</td>
      <td>first third 1900s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.229.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DAE4CA9BF61B0B35AB5BFDC914C3C7F3EB84F493D66249E1D&s=21&se=1391664542&v=&f=1956.229.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>143</th>
      <td>Reclining Nude Woman</td>
      <td>Emil Fuchs (American, 1866-1929)</td>
      <td>1926</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.204</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9FD37BAFC27E9FE1635F50E9C556153F9A4020412049D6D40BAB6ACAFAB9887B&s=21&se=1049999129&v=4&f=1926.204_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>144</th>
      <td>Reclining Nude Woman</td>
      <td>Emil Fuchs (American, 1866-1929)</td>
      <td>1926</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.205</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9FD37BAFC27E9FE1266BC01D2E6DFB79DFA038550A6389F5AC466C00B6D8B6D9&s=21&se=1049999129&v=4&f=1926.205_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>145</th>
      <td>Medallion, "Dancing Nude"</td>
      <td>NaN</td>
      <td>1800's</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1971.1048</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=6F529B5740B10182AEBC4D4BB019ADB4A255F2F70E7D95DA7DA5627C3C55B994&s=21&se=1135697452&v=&f=1971.1048_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>146</th>
      <td>Nude Study of an Old Man</td>
      <td>Henri Lehmann (French, 1814-1882)</td>
      <td>before 1852</td>
      <td>France</td>
      <td>In preparation for executing largescale paintings, artists often produced studies of individual components of their compositions in order to work through their ideas. Lehmann made this figural study for a mural in the old Hôtel de Ville in Paris, destroyed by fire during the Commune in 1871. Here, the artist explored how to render deep shadows on the nude male body.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.44</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=3AEF62CDCF453215D692B7B08BDFB9EC190E3F96CCDEEDA5CC9B8BCF260810A0&s=21&se=2006672921&v=1&f=2018.44_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>147</th>
      <td>Nude Man with Raised Arms</td>
      <td>Egon Schiele (Austrian, 1890-1918)</td>
      <td>1911</td>
      <td>Austria</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1959.304.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504B0F9D5EEA224F9A4FC6A1358F54D517BAE7F4F442CE87DB49&s=21&se=1674154316&v=3&f=1959.304.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>148</th>
      <td>Nude Lying Down</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.262</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233CFB35128814005BF342DB15584A4E1CC4979EA36FA158155E&s=21&se=1065507836&v=&f=1972.262_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>149</th>
      <td>Untitled (Nude Woman)</td>
      <td>Ernest Fiene (American, 1894-1965)</td>
      <td>1928</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1998.137</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=6C7D896C051051E8361C086B2BAEDFFBF7A6AB6B524F0ADBE2870CE412861BCC&s=21&se=83547869&v=4&f=1998.137_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>150</th>
      <td>Reclining Nude, No. 1</td>
      <td>Charles Frederick Ramus (American, 1902-1979)</td>
      <td>1930</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1930.154</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B5249C5C025392A4B022303FB820B1F8D6C6093DF28D5EA12410649A70479B92&s=21&se=1049999129&v=&f=1930.154_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>151</th>
      <td>Camera Work: Study [Nude]</td>
      <td>René Le Bègue (French, 1857-1914)</td>
      <td>1906</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.16.d</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9C187DB24CE475690B52C4D9ED0C2B76BB4222EA29A17C13FB5A067372A069F9&s=21&se=83547869&v=2&f=1995.199.16.d_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>152</th>
      <td>Crouching Male Nude</td>
      <td>NaN</td>
      <td>first half 1900s</td>
      <td>England(?)</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.272</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=79064E36F3F00558626C47B4D0F9A3AC77CED7DDC9BCDD563ECED9DF5176252F&s=21&se=1755401958&v=5&f=2003.272_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>153</th>
      <td>Nude Woman with Towel, Standing</td>
      <td>Edgar Degas (French, 1834-1917)</td>
      <td>1891-92</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1942.84</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=96B479B259297BE956941A7D7F75EEE5AE5462A89A12AAE85D14F1BEFC4F10D7&s=21&se=1031267696&v=3&f=1942.84_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>154</th>
      <td>Female Nude Standing</td>
      <td>David Karfunkle (American)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.169</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E88C03607559CAD1BDD33867AE709E1828835901D88446307&s=21&se=1049999129&v=&f=1923.169_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>155</th>
      <td>Nude Woman Seated</td>
      <td>Abel George Warshawsky (American, 1883-1962)</td>
      <td>1919</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1942.885</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C59B1D05F3D2EEAA278162181382794122313579B8102F10E&s=21&se=1031267696&v=2&f=1942.885_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>156</th>
      <td>Nude Woman Seated (verso)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1953.612.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA77316421865ACED87B1809693102CF19C60997D4741C5B2CD1A70290CA&s=21&se=1031267696&v=&f=1953.612.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>157</th>
      <td>Standing Male Nude</td>
      <td>NaN</td>
      <td>first half 1900s</td>
      <td>England(?)</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.271</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=79064E36F3F0055890C62FE1DF6CCFCB1EB8D5E7EAD80D488EF75E81E3D71ACD&s=21&se=1755401958&v=3&f=2003.271_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>158</th>
      <td>Study of a Seated Nude Female Model Drawing</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>c. 1888-1898</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>In this surprising drawing, a nude figure abandons her role as model in favor of drawing. The passive nude has become active; she turns away from the viewer, absorbed in her own artistic pursuit.</td>
      <td>https://clevelandart.org/art/2010.250</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5D0F103AF8C2334218D68A123A5CF4C75B9A3CF44500C79B2B054CBC7A2C37B5&s=21&se=1171617854&v=&f=2010.250_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>159</th>
      <td>Two Nude Figures</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1909</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1973.12</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72B5EAEA77CF6E0C1F151B489AFD7151F9FF3C6A5D5D4FC444&s=21&se=1065507836&v=3&f=1973.12_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>160</th>
      <td>Nude Figure in a Niche</td>
      <td>Théodore Rivière (French, 1857-1912)</td>
      <td>c. 1870 - 1912</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.119</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CBFA069969C7904756B70F68690ED5F22C42B32A289ACB7809&s=21&se=1329612074&v=&f=1978.119_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>161</th>
      <td>Female Nude with Outstretched Arms</td>
      <td>William McGregor Paxton (American, 1869-1941)</td>
      <td>c. 1913</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.269</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=976013C7DEFC9D5619080C47644E3A07AFF4258695807BA5054D3CC705EDC399&s=21&se=1755401958&v=&f=2003.269_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>162</th>
      <td>Eva, Study of a Nude</td>
      <td>Emil Fuchs (American, 1866-1929)</td>
      <td>1913</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1916.899</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDEFD897FE0BF958CDDD00B4D7CA885E0340814480A81AFF9F8&s=21&se=754529343&v=&f=1916.899_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>163</th>
      <td>Two Nude Figures</td>
      <td>Tsugouharu Foujita (French, 1886-1968)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1944.420</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A9D32762E79BC68694CB522630535389FCA06F3D7EB3E4DDD6570DF2EABB2476&s=21&se=1031267696&v=&f=1944.420_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>164</th>
      <td>Nude Study, Woman Lying Prone</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.616</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E10945D62AC633AC6DFF5D21449F20566C19E8859B566B606E09&s=21&se=1049999129&v=4&f=1936.616_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>165</th>
      <td>Battle of the Nudes</td>
      <td>Antonio del Pollaiuolo (Italian, 1431/32-1498)</td>
      <td>1470s-1480s</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>This engraving is one of the earliest Renaissance prints to portray the nude male body in action. Pollaiuolo’s grimacing warriors appear like clones in different poses. The print may have functioned as a model for workshop apprentices studying human anatomy while learning to draw; however, the artist’s Latin signature suggests it also had an audience educated in literature. Art historians remain uncertain whether Pollaiuolo intended to depict a particular story or historical event. It is possible he created a deliberately ambiguous allegory that would appeal to patrons interested in interpreting symbols. For example, the continuous chain shared by the two central men could refer to an ancient idea that the body is the chain of the soul, only to be released in death.</td>
      <td>https://clevelandart.org/art/1967.127</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2FFF96D56F26D7BB4E7A87AB48DD162F99E27E4782B9AD7131415443BD67C4A3&s=21&se=172172604&v=&f=1967.127_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>166</th>
      <td>Nude, Flying Figure</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.113</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C402C6AA14BD15CC8FD138F64C6353CC0A1557760B6EA856D&s=21&se=1523204814&v=&f=1978.113_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>167</th>
      <td>Standing Nude Model</td>
      <td>Otto H. Bacher (American, 1856-1909)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1983.244</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E5BE6EE844750D03700F6FC5DEC680C21EA393444C2D11D21&s=21&se=933923230&v=&f=1983.244_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>168</th>
      <td>Nude Woman Standing, Drying Herself</td>
      <td>Edgar Degas (French, 1834-1917)</td>
      <td>1891-1892</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.361</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=33B458E6429A5D5973F8EB1006477BEAA2DAD0E8F704A2D39CAEF6F214EDA071&s=21&se=1031267696&v=3&f=1954.361_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>169</th>
      <td>Nude No. 2 (Seated Female Nude, Facing Right)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1940.79</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186203DBD9563C523D30CFFDC82A05EB0056AAB7DA8F0EDA102&s=21&se=1031267696&v=&f=1940.79_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>170</th>
      <td>Study - Female Nude Standing</td>
      <td>Abel George Warshawsky (American, 1883-1962)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.167</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8CF46D0C9092643672E9A0821ECD824367200B8D8274A111FD&s=21&se=1049999129&v=2&f=1923.167_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>171</th>
      <td>Male Nude Study</td>
      <td>Mark Tobey (American, 1890-1976)</td>
      <td>1952</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.654</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186E48D46E1A76002506E79DE433E461A7310DBA584C78339BE&s=21&se=1031267696&v=&f=1954.654_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>172</th>
      <td>Camera Work: Nude</td>
      <td>Alice M. Boughton (American, 1866-1943)</td>
      <td>1909</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.26.e</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2839946BA949AFAA32EE22BABD97B5469ACB2AC7EF144AAA337AAF93F831E54F&s=21&se=83547869&v=2&f=1995.199.26.e_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>173</th>
      <td>Reclining Female Nude</td>
      <td>William-Adolphe Bouguereau (French, 1825-1905)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.304</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D926A625DCCAA59E438DBD88EFD32DD6BCBDD1661F074556140F&s=21&se=1406770801&v=&f=2016.304_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>174</th>
      <td>Rounded Nude in Water</td>
      <td>Henriette Grindat (Swiss, 1923-1986)</td>
      <td>1969</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.586</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=917852C60E2556A364237974BC8C103FC94BEB4E8AEB1B9AE86BBB1CC1ED064A&s=21&se=1824257385&v=1&f=%5Cd7930%5Cu168879308%5C2018.586_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>175</th>
      <td>Eight Nude Children at a Well</td>
      <td>Heinrich Aldegrever (German, 1502-1555/61)</td>
      <td>1539</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1922.126</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=BDFDD69D477608B15EE284B2D7712463ED067E919394B306AD6232A10CEEE187&s=21&se=1049999129&v=4&f=1922.126_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>176</th>
      <td>Camera Work: Nude</td>
      <td>Clarence H. White (American, 1871-1925)</td>
      <td>1908</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.23.j</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9C187DB24CE475692A7DDB33C90DCC11DD61BA630E8D5C5F2627EEFD5995460C&s=21&se=83547869&v=2&f=1995.199.23.j_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>177</th>
      <td>NOVA Portfolio: Nude</td>
      <td>NOVA, New Organization for the Visual Arts, Inaugural Exhibition Print Portfolio</td>
      <td>1973</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2005.301.8</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D32CDCDF4C47E50FFEF0C1ED0C7ECE3AF9880D7A8FA58AC4F3A0E51719DFA537&s=21&se=1755401958&v=5&f=2005.301.8_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>178</th>
      <td>Camera Work: Nude - A Study</td>
      <td>Frank Eugene (German, 1865-1936)</td>
      <td>1910</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.31.m</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2839946BA949AFAA0D06E16AA667037A52F013580437FC11B33085364E39C8D5&s=21&se=83547869&v=2&f=1995.199.31.m_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>179</th>
      <td>La Danseuse - A Study of the Nude</td>
      <td>James McNeill Whistler (American, 1834-1903)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.630</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C7480607BFD52B2462AA9ACD761AEB3BB69528CA32D590A31BCF815E&s=21&se=1031267696&v=3&f=1941.630_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>180</th>
      <td>The Kitchen Table Series: Untitled (Nude)</td>
      <td>Carrie Mae Weems (American, b. 1953)</td>
      <td>1990</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2008.116.18</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=168C18B5D078DF852FA95CF4DB1FB398DC3C3642C4D7DDF9E29222654BC66ED3&s=21&se=1828349793&v=2&f=2008.116.18_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>181</th>
      <td>Figurine of a Nude Woman</td>
      <td>NaN</td>
      <td>400-200 BC</td>
      <td>Greece</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1927.7</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1E0711E1129993B7E62A687EB8228E1F5131E388173CB3AD8792D1DB9CBB863D&s=21&se=2130744020&v=4&f=1927.7_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>182</th>
      <td>Nudes</td>
      <td>Jacques Villon (French, 1875-1963)</td>
      <td>1914</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1973.209</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA77316421866716B81331752137D8AAFEE2DB8BC36D42DB6A4E10AADADB&s=21&se=1065507836&v=&f=1973.209_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>183</th>
      <td>Study of a Nude Woman Seated</td>
      <td>Boris (Hungarian)</td>
      <td>None</td>
      <td>Hungary</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1931.227</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5183C4D25C7FECF7E7976D69A5C3EB54AB0E69681F3FF38941F08806D201BF0C&s=21&se=1049999129&v=3&f=1931.227_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>184</th>
      <td>Nude Study, Classic on a Couch</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.595</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E1095D67D94DF88BDBD8F8960F5333CD853CCE4DD34FAF391B60&s=21&se=1049999129&v=4&f=1936.595_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>185</th>
      <td>Female Nude, Seated, Three Quarter View from Front</td>
      <td>William Mulready (British, 1786-1863)</td>
      <td>1859</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>For Mulready, drawing was a lifelong preoccupation. These two sheets were drawn from models in the Royal Academy’s life school when the artist was 73. He was one of the Academy’s most devoted teachers, positioning the model for his students and then drawing alongside them. Closely observed and meticulously crafted, both drawings attest to Mulready’s consummate skill as a draftsman. The sensuality of the female nude and the weather-beaten face of the male model are as arresting today as they were in Victorian London.</td>
      <td>https://clevelandart.org/art/1978.53</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=95FCF11A0E49A29976F0AE7C198E6CFC861989D52A6430A64F1AD4981A1EEA8A&s=21&se=1329612074&v=3&f=1978.53_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>186</th>
      <td>Camera Work: Nude - The Pool</td>
      <td>George H. Seeley (American, 1880-1955)</td>
      <td>1910</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.29.f</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2839946BA949AFAA3EBA7F4663412C76A3B9B9019F9CD61024549F04C8802D4A&s=21&se=130305038&v=2&f=1995.199.29.f_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>187</th>
      <td>Half Nude Figure of a Man</td>
      <td>Robert Frederick Blum (American, 1857-1903)</td>
      <td>1800s</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1920.1311</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D844B35FED6A401F9881E1EE3A432885927058F7730776C3C&s=21&se=704959412&v=3&f=1920.1311_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>188</th>
      <td>Two Nude Men Beside a Tree</td>
      <td>Marcantonio Raimondi (Italian, 1470/82-1527/34)</td>
      <td>None</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.192</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=AF226C61E597B26C0C1BFE19FA8F7FA015EFE5BDD78ED00A35E607B2325CAC7B&s=21&se=1049999129&v=3&f=1923.192_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>189</th>
      <td>Evening, Nude on a Bed</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1921</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1935.276</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DFF77E9D569E693EC2402FAE1A759756913EAB4356D703B55&s=21&se=1049999129&v=3&f=1935.276_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>190</th>
      <td>Study for the Nude Youth over the Prophet Daniel (recto); Figure Studies for the Sistine Ceiling (verso)</td>
      <td>Michelangelo Buonarroti (Italian, 1475-1564)</td>
      <td>1510-11</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Universally considered one of the greatest artists of the Italian Renaissance, Michelangelo devoted four years to painting the vast ceiling fresco in the Sistine Chapel. This preparatory study portrays one of the 20 athletic male nudes, known as ignudi, who serve as supporting figures at each corner of the Old Testament scenes painted down the center of the ceiling. Michelangelo worked out the positioning of the ignudi in red chalk drawings before beginning to paint each section of wet plaster. The energy and monumentality of the figure in red chalk, whose body extends beyond the sheet, suggests the heroic athleticism of Michelangelo’s sculpture.</td>
      <td>https://clevelandart.org/art/1940.465</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CDE735E74C0ACE20F5C8A151D6A3ED909B3A4BC9DE5A9FC0DC3B120FC657A573&s=21&se=1031267696&v=1&f=1940.465_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>191</th>
      <td>Nude Woman with Mirror (Mädchen vor dem Spiegel)</td>
      <td>Karl Schmidt-Rottluff (German, 1884-1976)</td>
      <td>1914</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Karl Schmidt-Rottluff accentuated the material qualities of the woodblock used to make this print by leaving evidence of his gouging in the negative spaces and by emphasizing the simple, rough contours and flattened proportions of the female figure. While the traditional subject of a female nude in front of a mirror suggests an allegory of beauty or vanity, Schmidt-Rottluff’s nude avoids idealization. The model’s mask-like features and body decoration also reference tribal art, which Brücke artists admired at the Dresden ethnological museum, particularly the wooden relief carvings from Palau, Micronesia, that show similar forms and markings.</td>
      <td>https://clevelandart.org/art/2016.607</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=69D17541278DB36B2BA80006091A25A825F6CD1ED2DB2F9A8777E11F06FD9069&s=21&se=641743206&v=&f=2016.607_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>192</th>
      <td>The Little Nude Model Reading</td>
      <td>James McNeill Whistler (American, 1834-1903)</td>
      <td>1890</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.624</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C748060791211FD98F480610D837F378D32C568424FC0D77D18FECCE&s=21&se=1031267696&v=3&f=1941.624_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>193</th>
      <td>The Little Nude Model Reading</td>
      <td>James McNeill Whistler (American, 1834-1903)</td>
      <td>1890</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1942.996</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2164CEA9C74806070F4E1625CDB1C8C9D162556A3E4FCC62D5D86534E53C84D7&s=21&se=1031267696&v=3&f=1942.996_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>194</th>
      <td>Nude on a Striped Rug</td>
      <td>Philip Pearlstein</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1974.261</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4B241336CD74C170AD889937B2FC779FA600D715986A0FA91B00C43ED74F86B7&s=21&se=1523204814&v=2&f=%5Cd7212%5Cu2972128%5C1974.261_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>195</th>
      <td>Camera Work: Nude - A Child</td>
      <td>Frank Eugene (German, 1865-1936)</td>
      <td>1910</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1995.199.31.k</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2839946BA949AFAA48734AA51A95D6A0FB763C03ACEE7EF1A35BE5D77706BE00&s=21&se=83547869&v=2&f=1995.199.31.k_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>196</th>
      <td>Seated Female Nude (Emma Story Bellows)</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>after 1910</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Bellows achieved fame as a young man for his powerful boxing scenes and images of urban life, but he also produced sensitive drawings and paintings of women. Bellows married Emma Story in 1910.</td>
      <td>https://clevelandart.org/art/1950.453</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB4B17197B7B25E25BDBC43AFB9969266AAD0CFB992167088B&s=21&se=1031267696&v=6&f=1950.453_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>197</th>
      <td>Nude Woman Standing, Side View</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1916</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1937.487</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D03D33F6C2876CF61470D1971925509545BF6B7E2DD33BCF2&s=21&se=1049999129&v=&f=1937.487_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>198</th>
      <td>Female Nude Seated on a Low Bench</td>
      <td>John Singer Sargent (American, 1856-1925)</td>
      <td>1919</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1924.231</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA7731642186E68119908AFB3EB80CEA6590BC1EDD7C83077D6D2686B4CC&s=21&se=1049999129&v=&f=1924.231_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>199</th>
      <td>Study of a Nude Woman Standing</td>
      <td>Boris (Hungarian)</td>
      <td>None</td>
      <td>Hungary</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1931.226</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5183C4D25C7FECF725FC4FCAEF9EFC6CF80A754A046346357F04FED104FA0B1E&s=21&se=1049999129&v=3&f=1931.226_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>200</th>
      <td>Nude Woman Seated on a Sofa</td>
      <td>Jean Veber (French, 1868-aft 1907)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1941.536</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5EBF27417EA07FF65B64C8CFB64FB616DBE88D5EFE11A94BEF&s=21&se=1031267696&v=4&f=1941.536_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>201</th>
      <td>Study - Female Nude Seated on a Stool</td>
      <td>Abel George Warshawsky (American, 1883-1962)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.166</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8CB88D5A3E19A3C3FCF608099EF4D11D202C3CCA2F0F77167B&s=21&se=1049999129&v=2&f=1923.166_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>202</th>
      <td>Seatetd Nude Woman from the Back</td>
      <td>Jean Louis Forain (French, 1852-1931)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1928.751</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A9D32762E79BC6864B94D2A1BBE1351E60B27198F5281794858CCE6ED80CFB35&s=21&se=1049999129&v=3&f=1928.751_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>203</th>
      <td>Untitled</td>
      <td>Karl F. Struss (American, 1886-1981)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2012.314</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=242775E0B50F25E8D663FEA72C66C7F8986C5B02703A70F44375685D46D1BE7D&s=21&se=1689619227&v=&f=2012.314_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>204</th>
      <td>Rubber Stamp Portfolio: Shiny Nude</td>
      <td>A. Colish Press</td>
      <td>1977</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2008.211.12</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=7E5DE8EAA79ADB678DD83747E8EF9ADE5C3F4F8EC11213D9EB21E21AD3E614A7&s=21&se=1285208684&v=&f=2008.211.12_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>205</th>
      <td>Morning, Nude on Bed, First Stone</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1921</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.557</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D2570262F1CC3FE1E32E10E44B0874C91234C9671560BA8E6&s=21&se=1049999129&v=3&f=1936.557_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>206</th>
      <td>Photographs of a Standing Male Nude Model ("Joseph Smith")</td>
      <td>Thomas Eakins (American, 1844-1916)</td>
      <td>c. 1883</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Eakins’s earliest use of photography was as a visual aid to painting. In the early 1880s, he began photographic experiments that would allow him to create a more accurate and convincing rendering of anatomy and the human form in motion. Recognizing the assistance these visual reminders could provide the artist, Eakins and his circle began photographing a series of nude forms modeled by men, women, and children. These images, which he called the Naked Series, consist of seven contact-printed photographs mounted on cardboard strips. They show the models from the front, side, and rear posed in contrapposto positions or with their weight equally distributed. Eakins’s other photographic endeavors included a series on the figure in motion—photographs that were clearly used as studies for paintings—and more traditional portraits and figural groupings.</td>
      <td>https://clevelandart.org/art/1997.131</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE6E67032F06DF818E9D21D98B36F7774D049BB61077409DAF&s=21&se=2019037029&v=4&f=1997.131_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>207</th>
      <td>Female Nude Seen from the Rear</td>
      <td>William McGregor Paxton (American, 1869-1941)</td>
      <td>c. 1913</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.268</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=976013C7DEFC9D56AB6F965351FCF0C9E077DB4BF0B930E17D7D6A04173A53CA&s=21&se=1755401958&v=&f=2003.268_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>208</th>
      <td>Studies of a Male Nude (verso)</td>
      <td>John Singer Sargent (American, 1856-1925)</td>
      <td>1918-19?</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1997.256.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=91AEDF5376158FC0DFC3FED0458D74FAD6135B741EC906A00BDDEF42860C2DFE&s=21&se=83547869&v=&f=1997.256.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>209</th>
      <td>Laurels Number Four: Nude with Long Torso</td>
      <td>Laurels Gallery</td>
      <td>1948</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Milton Avery was a quiet, humble man whose art reflected his life. Inspired by personal experience and based on direct observation, his relaxed, intimate works depict a serene, ideal world. Landscapes evoke summer trips to the country and seaside; urban scenes depict family and friends. From apparently ordinary subject matter, Avery constructed innovative works in which simplified flat forms build perfectly balanced, satisfying compositions. \r\n\r\nBetween 1947 and 1948, Chris Ritter, director of the Laurel Gallery, New York, published four issues of Laurels, portfolios of prints, literature, and original calligraphy often on paper handmade by Douglass Howell. The museum's collection already includes the first two issues, with prints by Joan Miró, Stanley William Hayter, Reginald Marsh, and others. The five drypoints shown here were made for the fourth Laurels Portfolio. They encompass Avery's favorite motifs: landscape, seascape, and figure, all delineated with amazing economy. For example, the Nude with Long Torso is drawn with two horizontal lines that span the length of the plate. By subtly controlling the thickness of line or placement of a shape, a sense of roundness, weight, and a shallow space are created.\r\n</td>
      <td>https://clevelandart.org/art/2001.9.5</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=15206FCEB8516298AC521F0431C8F6246D1DC7E4365A0777292C43643D72A383&s=21&se=1755401958&v=5&f=2001.9.5_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>210</th>
      <td>Nude Study, Woman Stretched on Bed</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1937.474</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DD05F7EAE44EF5F785AEEBCD1E6C595DDA93BBFFED7A78BF1&s=21&se=1049999129&v=3&f=1937.474_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>211</th>
      <td>Nude Study, Woman Kneeling on a Pillow</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1935.246</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D83507C9A2F7B5FFB2A8D54738C5DA804EC626CC5DAF1F7D2&s=21&se=1049999129&v=3&f=1935.246_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>212</th>
      <td>Nude, Study for The Harem (Woman Washing Herself)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1905</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/435.1932</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=86723FDC492ACEDDEBAE5E71251829D10AD16AF588EE54DD6DA36701D699101A&s=21&se=329679043&v=3&f=435.1932_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>213</th>
      <td>Nude Child Seen from the Rear</td>
      <td>William McGregor Paxton (American, 1869-1941)</td>
      <td>1913</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.267</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=976013C7DEFC9D56FD85AD5C8DE31F7E9334764D67592B0A9D462FEEF87CF981&s=21&se=1755401958&v=&f=2003.267_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>214</th>
      <td>Nude Study, Girl Standing on One Foot</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1937.476</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D0E8B4C1275963677B6C9402D43667BD4CDEE2E68CF103511&s=21&se=1049999129&v=3&f=1937.476_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>215</th>
      <td>Morning, Nude on Bed, Second Stone</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1921</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.556</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D733D8512670F3EDB13A1C97036686E98D8D36A3716AE0A71&s=21&se=1049999129&v=3&f=1936.556_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>216</th>
      <td>Nude Study, Woman Lying on a Pillow</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.632</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E1098F57DDD882736CF60BA3CDA2C7AE2139BFDD19926EC731A6&s=21&se=1049999129&v=4&f=1936.632_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>217</th>
      <td>Verona Sketchbook: Female nude (page 27)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.aa</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE969983BDE7D562282CBF545A5414AAFB84998811121F5D3E6&s=21&se=1031267696&v=&f=1952.223.aa_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>218</th>
      <td>Nudes</td>
      <td>Cecil Buller (Canadian, 1886-1973)</td>
      <td>1920s</td>
      <td>Canada</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2013.398</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=33F6709B48627EE35B5CB324A73C5271782A6DBB70C475B155B13FEA0C2AC055&s=21&se=1400739526&v=&f=2013.398_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>219</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F88318034C64C677973B573AE98A37378C063B48C5924A481F0E1A4E343534A2&s=21&se=1065507836&v=&f=1972.53_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>220</th>
      <td>Academy of a Seated Nude Holding a Staff</td>
      <td>Edmé Bouchardon (French, 1698-1762)</td>
      <td>c. 1735/50</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1999.20</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9332AAE04CF4DB604ADB532938E3B97C403EA9FE4C2FA8F391BE4A6EABB03264&s=21&se=1755401958&v=3&f=1999.20_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>221</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.8</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72859641FC4AEE9AFE9ADCC98991339E74D591196177A44218&s=21&se=1065507836&v=6&f=1972.53.8_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>222</th>
      <td>Sketchbook, page 100: Nude Figure, Profile</td>
      <td>Ernest Meissonier (French, 1815-1891)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.344.ssss</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F276110D0AD483E5F237C63759EDC4F4B0D74730E3E5FA89267A0F&s=21&se=466550310&v=&f=1992.344.ssss_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>223</th>
      <td>11 Pop Artists, Vol. II: Nude</td>
      <td>Tom Wesselmann (American, 1931-2004)</td>
      <td>1965 (published 1966)</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.620</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=69D17541278DB36B46A727ABA1BD2947B9C1B5FF1C051534F59E3C9BE9F5319F&s=21&se=1296451997&v=&f=2016.620_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>224</th>
      <td>Man on his Back, Nude, second state</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.622</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E10938E7BD44C8C68B8B0ED003DD365BF56925DCEDC28F814BA6&s=21&se=1049999129&v=&f=1936.622_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>225</th>
      <td>Nude Woman Seated, Seen from the Back (recto); Nude Woman Seated (verso)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1953.612</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4067D3A7A2A1C210C10BF0D4AB54960903F58D9FD95539FE505A6DBF495A8ED4&s=21&se=1031267696&v=&f=1953.612_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>226</th>
      <td>Seated Female Nude from the Rear</td>
      <td>NaN</td>
      <td>first half 1900s</td>
      <td>England(?)</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.270</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=6C4006907967A5A28CD315F2BCD70B8BA9A89C7F02DB53709F8E0B09118F1914&s=21&se=1755401958&v=4&f=2003.270_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>227</th>
      <td>Sketchbook, page 60: Nude Male Study</td>
      <td>Ernest Meissonier (French, 1815-1891)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.344.ggg</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611971772B7F62E88CD9DAA753FD766608D8AFCB8708B01D612&s=21&se=466550310&v=&f=1992.344.ggg_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>228</th>
      <td>Nude Man with Raised Arms (recto) Seated Man (verso)</td>
      <td>Egon Schiele (Austrian, 1890-1918)</td>
      <td>1911</td>
      <td>Austria</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1959.304</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C780E6B995A7054CEFCF5FC681A86F8611C152013ACAB2010BD2EC52362B9C64&s=21&se=1674154316&v=&f=1959.304_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>229</th>
      <td>Standing Nude Bending Forward, third state\r\n</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1916</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.562</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D999BB7A4BE79A4F809E524FB2E46F653F90AF1E67BBABEF7&s=21&se=1049999129&v=&f=1936.562_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>230</th>
      <td>Le Cirque de l'étoile filante:  Nude</td>
      <td>Georges Rouault (French, 1871-1958)</td>
      <td>1938</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.290</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=40675183989CEBD80404E9E92DBB77CB420B85A7A1B130EF1AD6581BB9CF3118&s=21&se=1110187508&v=3&f=1956.290_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>231</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.2</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA723A75A765601E38C2533EA68238D0CF89805BB2092EF6FFED&s=21&se=1065507836&v=5&f=1972.53.2_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>232</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.7</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7209DC1669D77AA62D986EF72DF729343319B61918B6302E2F&s=21&se=1065507836&v=5&f=1972.53.7_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>233</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.5</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA728FA8E767546B9D959D651D5660472B3F758F062F96C819FE&s=21&se=1065507836&v=5&f=1972.53.5_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>234</th>
      <td>Study of Madame Marie Cantacuzène; Study of Standing Female Nude</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>c. 1883</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2010.243</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5D0F103AF8C23342E8F18421F6FCAEA98E8A749E6A5927B2B04947E32B96845E&s=21&se=800429483&v=&f=2010.243_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>235</th>
      <td>Nude Study, Girl Standing with Hand Raised to Mouth</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1937.486</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DCC3C7EA9D55AA0EB43CD5DD58702BC221E0B38D539ED2D1B&s=21&se=1049999129&v=3&f=1937.486_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>236</th>
      <td>Sketchbook, Spain: Page 22, Head of a Female Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1922</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.206.10.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D926F3A2DD41335FBD61B0DB4F87C88AAF2E993DA41E5EABBAB0&s=21&se=445065177&v=&f=2016.206.10.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>237</th>
      <td>Nude Woman Seated, Seen from the Back (recto)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1953.612.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA77316421868840AA796B50D0719FDBD67DF8BBA7B420966789C22A96E3&s=21&se=1031267696&v=&f=1953.612.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>238</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.3</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7291E1FA355C751AF806D88155D53E3CF4DCBEA23182AA1C67&s=21&se=1065507836&v=5&f=1972.53.3_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>239</th>
      <td>Nude Study, Girl Sitting on a Flowered Cushion</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.567</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70DFEA3F88E94D4AB4372EBC091D8A08B6A40BD6EDCD7A96DDE&s=21&se=1049999129&v=4&f=1936.567_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>240</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.6</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72E2DA7FD87729131A512B37AC7DF1AFED5B9B628753FAD5AC&s=21&se=1065507836&v=5&f=1972.53.6_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>241</th>
      <td>Sketchbook, Spain: Page 23: Studies of a Female Nude</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1922</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.206.11.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D9261D0F9851C78116B0778C78CDCD83C1A6833E0FE8112C7064&s=21&se=445065177&v=&f=2016.206.11.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>242</th>
      <td>Life Study, Nude Woman Seated With Folded Hands</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1936.607</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=04AD67C577E5E109A59B0E17F52055E0D4D45641169567D4A2A26B91FD970AB6&s=21&se=1049999129&v=&f=1936.607_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>243</th>
      <td>Sketchbook, The Dells, N° 127, page 092: Nude in Profile</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>c. 1919-1921</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 59 pages of graphite drawings and 16 watercolors made at various locations on Cape Ann, Massachusetts.</td>
      <td>https://clevelandart.org/art/1951.425.aa</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611480F9936836DD58BB38229491CEAB2EF4587B6837A114D86&s=21&se=1031267696&v=&f=1951.425.aa_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>244</th>
      <td>Verona Sketchbook: Female nude looking over left shoulder (page 74)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.vvv</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE9CF53F104DDF457B99483FB200314487501B708A3D3C68508&s=21&se=1031267696&v=&f=1952.223.vvv_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>245</th>
      <td>Bauhaus Portfolio III: Nude Woman Seated in Landscape with Farmhouse</td>
      <td>Heinrich Campendonk (German, 1889-1957)</td>
      <td>1920-1921</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2000.184</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=87A1FAC2FDC603F6358D7D3DE528FCCFEFC1400C41A7AAB278D381376D76B952&s=21&se=1755401958&v=4&f=2000.184_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>246</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.4</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7205CA3248F99B51933BF082147052FCC925089E24377080F3&s=21&se=1065507836&v=3&f=1972.53.4_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>247</th>
      <td>Sleeping Cat (recto) Torso of a Nude Woman (verso)</td>
      <td>Jane Poupelet (French, 1878-1932)</td>
      <td>first third 1900s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.229</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C780E6B995A7054CD2B1D83968973F36EE690408FAF987444342A0FFD7F28793&s=21&se=1391664542&v=&f=1956.229_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>248</th>
      <td>Nudes in a Landscape</td>
      <td>Jean-Jacques-François Lebarbier, called Le Barbier L’Aîné (French, 1738-1826)</td>
      <td>1781</td>
      <td>French</td>
      <td>Le Barbier’s watercolor reflects the taste for Neoclassicism in late 18th-century France. In addition to the representation of nude figures, the artist’s admiration for antique art and architecture is also seen in the statuesque group of robed women standing in the central middle ground and the classicizing round temple on the right.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.50</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=855247B1318B8036BC3C7EEA0B769EDD7A7476684207FFCC8ECD381DCCACFC2A&s=21&se=1789656799&v=1&f=2018.50_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>249</th>
      <td>Verona Sketchbook: Nude male torso with drapery (page 16)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.p</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE99B4AD7331CA7E29AF5BA213C567789398E2D651E2D5D9669&s=21&se=1031267696&v=&f=1952.223.p_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>250</th>
      <td>Three Male Nudes</td>
      <td>Domenico Beccafumi (Italian, 1486-1551)</td>
      <td>c. 1540-1547</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Chiaroscuro drawing, a common technique in the 1500s, is usually done on a medium toned sheet with dark colors indicating the outlines and shadows and white pigment used for the highlights. Prints imitating these drawings were popular at the time.</td>
      <td>https://clevelandart.org/art/1958.313</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C88D54049EE977BC7FCB132AC695ACEC7C720105A5597DA6E0E3B2E2BF62A0BC&s=21&se=1674154316&v=4&f=1958.313_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>251</th>
      <td>Verona Sketchbook: Nude with head and right arm (page 36)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.jj</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE9323CED3BE342D672A3D6B49526A55BEA8074819085D1E43B&s=21&se=1031267696&v=&f=1952.223.jj_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>252</th>
      <td>Sketches of Seated Woman in Kimono (recto) Standing Nude (verso)</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>1914</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1954.830</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C780E6B995A7054C71A6A0E7BDBFB93B4B133BFACD0309675F16E1BFC1E1EB64&s=21&se=1031267696&v=&f=1954.830_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>253</th>
      <td>Nude Study, Girl Sitting on a Flowered Cushion</td>
      <td>George Bellows (American, 1882-1925)</td>
      <td>1923-1924</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1965.257</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=84C232B66150D70D602D4A86B472AC68ACA6574E63E01B555C619F0049DF7BEA&s=21&se=1279503776&v=5&f=1965.257_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>254</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.1</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA727380EB0EDEAC9FE67C85BC65DE6C1626E08EE39C88403768&s=21&se=1065507836&v=5&f=1972.53.1_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>255</th>
      <td>Two Nude Women (A series of progressive proofs)</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1945-1946</td>
      <td>Spain</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1972.53.9</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA72061334899E89D7F9D94D23F1F2717A398E71EBD62EB6BF15&s=21&se=1523204814&v=5&f=1972.53.9_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>256</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 133: Nude Walking</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.fff</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F2761104106C52A5478966881638F6C4866635C15C749800F33F38&s=21&se=1031267696&v=&f=1951.424.fff_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>257</th>
      <td>Verona Sketchbook :Male nude with upraised right arm (page 32)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.ff</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE93F2364DB1E069FEC68C365871296582932D11DAFD0154DA8&s=21&se=1031267696&v=&f=1952.223.ff_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>258</th>
      <td>Verona Sketchbook: Male nude from back (page 9)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.i</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE9DB4769E6713492185780B293B7A0D64D7656008BF5C1DDA8&s=21&se=1031267696&v=&f=1952.223.i_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>259</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 170: Female Nude</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.sss</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611091184D58722DF96056BAA647D8E0A47D51A86571F033082&s=21&se=1031267696&v=&f=1951.424.sss_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>260</th>
      <td>Study No. 2 (Standing Male Nude Seen from Back)</td>
      <td>William A. Van Duzer (American)</td>
      <td>1940</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1940.80</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C1403CA41D5FCED698C44C51F78251B262218BB786ABCCD60&s=21&se=1031267696&v=2&f=1940.80_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>261</th>
      <td>Two Male Nudes</td>
      <td>Louis de Boullogne (French, 1654-1733)</td>
      <td>1710</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Drawing the male nude from a live model was the cornerstone of artistic training at the Royal Academy of Painting and Sculpture in Paris. Louis de Boullogne was a professor there the year he made this drawing, and it may well record the pose he set during one of the life classes he supervised. The posing of two models together was established as a monthly practice at the Academy in 1706 and normally occurred during the last week of the month. The use of two men instead of one allowed the professor to suggest a narrative more easily. Here, their interaction implies a warrior dying in battle being supported by a companion.</td>
      <td>https://clevelandart.org/art/2008.339</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B69E2CB8AE85872210C8201394A80FB6097F557FF6AB7D0925077952580E0365&s=21&se=1579770806&v=2&f=2008.339_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>262</th>
      <td>Studies of a Soldier Drinking, for Gassed (recto); Studies of a Male Nude (verso)</td>
      <td>John Singer Sargent (American, 1856-1925)</td>
      <td>1918-19</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>In 1919, Sargent exhibited a large painting at the Royal Academy of Art in London called Gassed. The Dressing Station at Le Bac and on the Doullers-Arras Road. The British War Memorial Committee had commissioned the work from him as a way of honoring the sacrifices of World War I. The subject was based on a scene the artist actually witnessed during his visit to battlefields in France in 1918. This drawing is a study for one of the soldiers in the painting, which now hangs in the Imperial War Museum in London.</td>
      <td>https://clevelandart.org/art/1997.256</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F81105D8A913E556ABEDE618FF1456C5FB1EB4ACA0CC0C17D873F91E0B6B9B17&s=21&se=83547869&v=&f=1997.256_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>263</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 169: Female Nude</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.rrr</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F2761115A7A141628B97775788F9CCDED77BB8419B14F8C9AD2E7E&s=21&se=1031267696&v=&f=1951.424.rrr_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>264</th>
      <td>Study of a Female Nude (possibly for an unrealized allegorical painting) (recto)</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>1800s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2010.249.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5D0F103AF8C2334297695ECD49F42115EB7EDBAA1278987F82F4F4F52858BED0&s=21&se=1171617854&v=&f=2010.249.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>265</th>
      <td>Untitled, From the Series, The Female Figure (Nude in Profile with Arms Outstretched)</td>
      <td>Karl F. Struss (American, 1886-1981)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2011.367</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1A6140C726E24AA4440B4878048055F4394124934E95122B65F8DA90D318931A&s=21&se=1053970089&v=&f=2011.367_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>266</th>
      <td>Untitled, From the Series, The Female Figure (Standing Nude With Drapery)</td>
      <td>Karl F. Struss (American, 1886-1981)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2011.368</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1A6140C726E24AA4CDC9940B5916ACB850AC1E2C59A93C0F705D830BD52C6784&s=21&se=1502064348&v=&f=2011.368_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>267</th>
      <td>The Wood Engravings of Leonard Baskin:  Number 2, Three Nude Men</td>
      <td>Leonard Baskin (American, 1922-2000)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1962.430.2</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=AB88F68620E712630DDE2F3451D6B5B60797C8C61D038FCFA9F73216ECFBD8BB&s=21&se=1674154316&v=&f=1962.430.2_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>268</th>
      <td>Reclining Female Nude (recto); Various Sketches of Figures and Plants (verso)</td>
      <td>Anonymous</td>
      <td>19th century</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1989.242</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4067D3A7A2A1C210E9076A9BF4DAF4A82E0E152A87B199D22C7F717CB7B9C59F&s=21&se=1652408849&v=&f=1989.242_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>269</th>
      <td>Verona Sketchbook: Male nude head and shoulders from behind (page 62)</td>
      <td>Francesco Lorenzi (Italian, 1723-1787)</td>
      <td>1760</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1952.223.jjj</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1051DC451CFD0BE9493D52719928568305337414F03E659FF2E80FD182EC42F6&s=21&se=1031267696&v=&f=1952.223.jjj_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>270</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 065: Nude Female seen from behind</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.aa</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611756644FFC7F242C2597D34DCDFDC7160C449AB5D983A48B8&s=21&se=1031267696&v=&f=1951.424.aa_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>271</th>
      <td>Studies of Female Nudes</td>
      <td>Henri Fantin-Latour (French, 1836-1904)</td>
      <td>c. 1895</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1962.399</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932E5475DA8ED39288E5879B53366925682494F7661525F63134&s=21&se=1674154316&v=&f=1962.399_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>272</th>
      <td>Study of a Standing Male Nude, with a Study of Head in Three-Quarter Profile</td>
      <td>Cecco Bravo (Italian, 1607-1661)</td>
      <td>c. 1640</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.288</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=48E878EBDE2BBEE4A3635658000CAE345650F89755B4990B1B339CD0FE6A0851&s=21&se=1755401958&v=3&f=2003.288_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>273</th>
      <td>Small Sword</td>
      <td>NaN</td>
      <td>c.1650-1660</td>
      <td>Netherlands</td>
      <td>None</td>
      <td>Despite its small size the hilt and handle of this sword include numerous details, both figurative and geometric. Look for men on horseback, nude women, and a battle scene.</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1916.1093</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD1B948468E95FCF850E74F017B3F843C510E8BCF8810D1C644&s=21&se=156361197&v=3&f=1916.1093_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>274</th>
      <td>Sketchbook, Spain: Page 47, Studies of Nude Women, Flower, and a Seated Man with a Cap</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1922</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.206.23.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D92642D60959C0D500CC307430E4CF5F24E3E5CBEC97E9E3B9B7&s=21&se=445065177&v=&f=2016.206.23.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>275</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1963</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.576</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>276</th>
      <td>Nude Woman on a Broomstick (from Advertisement Series for Coloured Fabric, Beit & Co., Hamburg)</td>
      <td>Georg Tippel (German, 1875-1915)</td>
      <td>None</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2014.357</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E80CAAF0F88B5AA9A9AFCD483767CCF9A03ED60A87DBD99BAE3281313C31CE21&s=21&se=1778462870&v=&f=2014.357_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>277</th>
      <td>Sketchbook, Spain: Page 31: Sketch of a Nude Woman Holding a Pitcher, c. 1922</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1922</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.206.15.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D926048A5DF9BC9CE0F8CBF9781DA90DB94DDB73F05BF23118AA&s=21&se=445065177&v=&f=2016.206.15.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>278</th>
      <td>Two Nudes</td>
      <td>Erhard Schön (German, c. 1491-1542)</td>
      <td>None</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1923.260</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7273153BFE5FCE50761997976D550E9EC2DD1DC12D37BCF9F8&s=21&se=1049999129&v=3&f=1923.260_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>279</th>
      <td>Diary: March 5th '87, at 2-12-4 Kikkodai Kashiwashi (a)</td>
      <td>Tetsuya Noda (Japanese, 1940-)</td>
      <td>1987</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>Noda photographs his surroundings to record personal experiences at a specific time and place, creating a visual diary. Here, he emphasized the monotonous quality of this inhospitable urban scene, devoid of figures, by printing the image from a woodblock on a solid white background. The result is a generalized, rather than personal, evocation of memory that underscores the dehumanized nature of modern life.</td>
      <td>https://clevelandart.org/art/1996.258</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=64F4B2219FEC97ED9844FE0C606C0A13A8177D7BF138BF21D46DF69E25194766&s=21&se=686303180&v=5&f=1996.258_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>280</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 031: Female Nude seen from the back</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.j</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611E99D98E48AB11F1B81A301B41C55FBB3E98A9323F97E7C91&s=21&se=1031267696&v=&f=1951.424.j_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>281</th>
      <td>Dish: The Goddess Anahita</td>
      <td>NaN</td>
      <td>400-600</td>
      <td>Iran</td>
      <td>None</td>
      <td>None</td>
      <td>Anahita, the Zoroastrian goddess of water and fertility, was one of the last of the ancient tradition of Near Eastern nude mother goddesses.</td>
      <td>https://clevelandart.org/art/1962.295</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD1F01A2039CD9F8BCEF44D160DD5341F4FB0EEEB39FA47E0B9&s=21&se=418730041&v=5&f=1962.295_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>282</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1963</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.570</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>283</th>
      <td>Sketchbook, Spain: Page 30: Sketch of a Nude Woman Holding a Pitcher, c. 1922</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>c. 1922</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.206.14.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D9266F93EF9286D7E631F5FB83E4A9C4BA1BAB6ED47140F6C767&s=21&se=445065177&v=&f=2016.206.14.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>284</th>
      <td>Grandmother with Louise, Nude Seated on the Floor (Grand-Mère et Louise Nue Assise par Terre)</td>
      <td>Suzanne Valadon (French, 1865-1938)</td>
      <td>1910</td>
      <td>French</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.255</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=330DC049D83B1B3D8C812B67626A58AF3B3A217781978EA75540E50876C74295&s=21&se=1789656799&v=1&f=%5Cd3983%5Cu133339830%5C2016.255_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>285</th>
      <td>St. John the Evangelist</td>
      <td>Charles de La Fosse (French, 1636-1716)</td>
      <td>c. 1700-1702</td>
      <td>France</td>
      <td>This drawing is a study for a fresco in the Parisian church of Les Invalides. The structure's four pendentives are each decorated with images of the Four Evangelists, and the Cleveland sheet relates to the design for St. John. La Fosse developed his ideas in numerous preparatory studies before beginning the final painting. He portrayed the religious figures with power and exuberance, imbuing his decorative scheme with unprecedented vigor. Drawings by La Fosse are difficult to find in the United States, and this sheet is an exceptional example of the artist's <em>trois crayons</em> (three-chalk) technique of using black, red, and white chalk together. The chalks are subtly blended, creating the effect of billowing drapery rather than stiff linearity.</td>
      <td>This drawing was used primarily used by La Fosse to study the drapery that covers St. John's body. A nude drawing of the same figure is in Le Havre, France.</td>
      <td>This drawing is a study for a fresco in the Parisian church of Les Invalides. La Fosse portrayed the religious figures with power and exuberance, imbuing his decorative scheme with unprecedented vigor. Drawings by La Fosse are difficult to find in the United States, and this sheet is an exceptional example of the artist's trois crayons (three-chalk) technique of using black, red, and white chalk together. The chalks are subtly blended, creating the effect of billowing drapery rather than stiff linearity.</td>
      <td>https://clevelandart.org/art/1956.602</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BC112F596BE95DF7FBEC4881A46378D18DA4AA6AAB045DB99&s=21&se=1391664542&v=&f=1956.602_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>286</th>
      <td>Dancing Nudes with Dog</td>
      <td>William Sommer (American, 1867-1949)</td>
      <td>c. 1925-30</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>In 1914, Sommer moved to the rural Brandywine, located halfway between Cleveland and Akron, where he converted an abandoned schoolhouse into a studio that became an important meeting place for modern artists, poets, and musicians. Seeking freedom from convention by immersing himself in nature, Sommer explained, "I believe art should be as spontaneous as the song of a bird."</td>
      <td>https://clevelandart.org/art/2004.45</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=378C0CABB291C95080BDB90DAEE0DE258AE21D0BBAE178AF476E5A7D434E106B&s=21&se=1755401958&v=4&f=2004.45_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>287</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1964</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.569</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>288</th>
      <td>Three Male Nudes</td>
      <td>Domenico Beccafumi (Italian, 1486-1551)</td>
      <td>second quarter 1500s</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Domenico Beccafumi was a Mannerist painter and sculptor who completed several religious and civic commissions in his home town of Siena. An experimental printmaker, Beccafumi made a chiaroscuro print, mixing engraving and woodcut on the same sheet. This print is a rare impression of the engraved plate alone.</td>
      <td>https://clevelandart.org/art/1958.314</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57450226973172660C663438E70935195DAEBB6FF2C692671E&s=21&se=1674154316&v=4&f=1958.314_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>289</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>None</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.407</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>290</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1964</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.571</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>291</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1963</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.574</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>292</th>
      <td>Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1966</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.421</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>293</th>
      <td>Cèze River Nudes, Bessèges</td>
      <td>Lucien Clergue (French, 1934-)</td>
      <td>1996</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1996.416</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FF154038BDAC7A63642842E4922D23A0692C2779D68AC6EB2A&s=21&se=83547869&v=3&f=1996.416_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>294</th>
      <td>Two Seated Nudes</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.109</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C23C4BB7C464DCC68619D871E1CABD9F6248F94B4AC448F71&s=21&se=1329612074&v=&f=1978.109_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>295</th>
      <td>Study of Four Female Nudes</td>
      <td>Max Weber (American, 1881-1961)</td>
      <td>1912</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.82</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=54198508A5919BE9CD1C71B150582D4D33B0A670EAAE3977B8A276C5113BE64C&s=21&se=1755401958&v=5&f=2003.82_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>296</th>
      <td>Two Nudes Bathing</td>
      <td>Sol Witkewitz (American, 1891-)</td>
      <td>1919</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1971.186</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C414B7AEB402D23E74345FB350264FDC7044672EDC4936116&s=21&se=1065507836&v=&f=1971.186_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>297</th>
      <td>Female Figurine</td>
      <td>NaN</td>
      <td>1200-900 BC</td>
      <td>Mexico</td>
      <td>This ceramic figurine—in the style of Tlatilco, an early village site in central Mexico—depicts a female with an enigmatic motif incised on the back of her head and substantial traces of red and white pigment. Since many figurines from the period depict females, modern interpreters usually connect them to fertility concerns. They also seem to testify to the importance of women among early Mesoamerican cultures.</td>
      <td>While many figurines are nude, they were likely dressed in perishable clothing and accessories.</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1990.140</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=37C443E0D334608A6831817055E12362256F775B79D123CFC939ED85C6C7714D&s=21&se=1198197559&v=&f=1990.140_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>298</th>
      <td>Study of a Female Nude (possibly for an unrealized allegorical painting) (recto); Studies of Drapery and Study of a Landscape (verso)</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>1800s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2010.249</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F81105D8A913E55683B4B3B5B659BE2989D5B2BF9434D0F378E5AD0992F9322C&s=21&se=800429483&v=&f=2010.249_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>299</th>
      <td>Sketch of Two Nudes (recto)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.214.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C2A234F7CC10F02F36B3EA58D8F09028D25D3DE629FEAE7C1&s=21&se=1391664542&v=&f=1956.214.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>300</th>
      <td>Two Nudes on Blue Coverlet</td>
      <td>Printed from 5 aluminum plates at Landfall Press</td>
      <td>1977</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2010.798</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=348C32465D6CA3C5889C6401CC157EDDDCAD8089344DF7E3749A379A2CA77B7A&s=21&se=1502064348&v=&f=2010.798_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>301</th>
      <td>Two Standing Nudes</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1978.110</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B1F45C239239233C83AC5C5F351015F95185282D1A7AD71A3CF4399AE769C011&s=21&se=1523204814&v=&f=1978.110_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>302</th>
      <td>In the Waves (Dans les Vagues)</td>
      <td>Paul Gauguin (French, 1848-1903)</td>
      <td>1889</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Painted at Pont-Aven in northwest France, this depiction of a nude figure throwing herself into the sea suggests a metaphor for a modern European woman forsaking civilization and abandoning herself to her natural, primitive instincts. The simplified lines and exaggerated colors, especially the contrasting green and orange, seem invented rather than observed from life. Exhibiting the painting at the Café Volpini in Paris in 1889, Gauguin established himself as a leader of the Symbolist movement in art.</td>
      <td>https://clevelandart.org/art/1978.63</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D0CD2D88DEAD73C3A08583E416B17442322C47D66F1583780895ED189C49FC7B&s=21&se=1329612074&v=&f=1978.63_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>303</th>
      <td>Sketchbook, page 92: Two Nudes</td>
      <td>Ernest Meissonier (French, 1815-1891)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.344.kkkk</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E879269A36768952F5E679AD6A110F959D51EEEC823E9CABB8240CF2F41F562&s=21&se=466550310&v=&f=1992.344.kkkk_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>304</th>
      <td>Allegory</td>
      <td>NaN</td>
      <td>possibly early 1500s</td>
      <td>possibly Northern Italy</td>
      <td>None</td>
      <td>None</td>
      <td>This object might have graced a man’s hat or the handle of a sword. The allegory has yet to be deciphered, but the heroic male nude chained in flames probably addresses masculine virtues of strength or stoicism.</td>
      <td>https://clevelandart.org/art/1968.29</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE856222EDE2B62469C25849F0DFCE550CEAAB8ADEE0A24BFF&s=21&se=23404858&v=4&f=1968.29_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>305</th>
      <td>Male Nude</td>
      <td>Hans Erni (Swiss, 1909-2015)</td>
      <td>1956</td>
      <td>Switzerland</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1966.168</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>306</th>
      <td>Sketch of Two Nudes (recto) Torso Sketch (verso)</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.214</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C780E6B995A7054C25A000F040262071EEEB358D79E252ECED319EFDCD902508&s=21&se=1391664542&v=&f=1956.214_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>307</th>
      <td>Red Nude</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1950</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1985.403</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>308</th>
      <td>Bowl</td>
      <td>NaN</td>
      <td>300-500</td>
      <td>Iran</td>
      <td>None</td>
      <td>None</td>
      <td>The decoration on this bowl--a vine with a little nude, accompanied by musicians, drinking wine from a rhyton--is related to the Hellenistic cult of Dionysos, or Bacchus, which was brought to Iran by the Greeks and became absorbed into the cult of Anahita.</td>
      <td>https://clevelandart.org/art/1963.478</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD1FDDE05EE14438589EC8F98465A9318B538395D3012AE7551&s=21&se=1674154316&v=6&f=1963.478_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>309</th>
      <td>Aphrodite</td>
      <td>NaN</td>
      <td>400-200 BC</td>
      <td>Greece</td>
      <td>This nude figurine was likely inspired by the Aphrodite of Knidos by Praxiteles, a work that spawned countless reproductions of varying sizes in different media. Created from inexpensive terra cotta clay, this Aphrodite might have been commissioned as a votive offering by a person of modest means.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1927.489</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CB670FEC2161B3E715F72406297A17547D929A68D287E1C41DA4E9BC96E2E942&s=21&se=1049999129&v=4&f=1927.489_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>310</th>
      <td>Sleep</td>
      <td>Jean Bernard Restout (French, 1732-1797)</td>
      <td>c. 1771</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>The tradition of painting nude male figures in a studio setting was the cornerstone of artistic practice, teaching artists to depict the human body in complex poses in order to create larger narratives. However, by the late 1700s, some artists began to see these studies as independent works of art. By adding the wings and the poppies, Restout transformed his study into a more specific subject, and he first exhibited the work in a privately organized exhibition in 1783 under the title of Morpheus, the god of sleep.</td>
      <td>https://clevelandart.org/art/1963.502</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C9F86FF91675F699FD6C6C28907018ECC86160FE3BB973AD4B500EB30333A7B7&s=21&se=1674154316&v=4&f=1963.502_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>311</th>
      <td>Bust of a Nude Woman</td>
      <td>Jacques-Antoine-Marie Lemoine (French, 1751-1824)</td>
      <td>1780</td>
      <td>France</td>
      <td>Lemoine was famous for his black chalk portraits like this example of a bust of a nude girl. Given the sitter’s individualized features, this image could represent a specific portrait; however, her identity is now lost to modern viewers. Below the portrait on the blue oval mount, the artist signed and dated the work to indicate his authorship.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2017.214</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>312</th>
      <td>Sketchbook- The Granite Shore Hotel, Rockport, page 001: Female Nudes with Notes</td>
      <td>Maurice Prendergast (American, 1858-1924)</td>
      <td>1905-10</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sketchbook contains 69 pages of drawings, a diary largely devoted to the state of the artist's health, and a three-page discourse on the nature of genius and ideas.</td>
      <td>https://clevelandart.org/art/1951.424.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611CAC3BD872306829EF0297B618F97DD0DF33BAECAEEE5F8C4&s=21&se=1031267696&v=&f=1951.424.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>313</th>
      <td>Athlete or Apollo</td>
      <td>NaN</td>
      <td>probably 100-1 BC</td>
      <td>Greece</td>
      <td>This bronze statuette depicts a nude male, perhaps a young athlete or the youthful god Apollo. Lost to time are objects he would have held in his hands, perhaps a bow and arrow. The weight-shift stance of the youth recalls the style of Praxiteles, master sculptor of the 4th century BC.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1927.170</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CB670FEC2161B3E7F9DF43ED75FF368646BB11FD3F3AF8F7F10DD4C9AA5FC4A1&s=21&se=1049999129&v=4&f=1927.170_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>314</th>
      <td>After the Bath (large version)</td>
      <td>Edgar Degas (French, 1834-1917)</td>
      <td>1891-1892</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>About 1876 Degas began to make lithographs with which he attained a rich tonality using only black and white. As with every medium in which he worked, Degas pushed lithography to new levels of expressiveness. In the 1880s and 1890s, Degas made many studies of a nude drying herself after a bath. He once wrote: "it is essential to do the same subject over again, ten times, one hundred times. Nothing in art must seem to be chance, not even movement."</td>
      <td>https://clevelandart.org/art/1970.285</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=33B458E6429A5D5964F3649977EE7722B379EAD7ACFF14C3E43FAB62CBB20367&s=21&se=31284912&v=7&f=1970.285_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>315</th>
      <td>Torso of Venus</td>
      <td>NaN</td>
      <td>1-200</td>
      <td>Roman</td>
      <td>As avid collectors of Greek art, ancient Romans valued sculptures for their aesthetic qualities. This torso of Venus was likely inspired by the popularity of Praxiteles's nude Aphrodite from the Greek city-state of Knidos on the western coast of Asia Minor. Displayed in a circular shrine, it became one of the most celebrated sculptures in classical antiquity.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.565</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1E0711E1129993B7C772020869E73E516BA071B13A477C74E7BD65DD24D63705&s=21&se=1049999129&v=6&f=1926.565_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>316</th>
      <td>The White Dam</td>
      <td>Raphael Gleitsmann (American, 1910-1995)</td>
      <td>1939</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Unable to afford tuition, Akron-based Gleitsmann befriended faculty and students at the Cleveland School (now Institute) of Art where eventually he was invited to take classes without registering. The White Dam, a fantastical image of an industrial complex that dwarfs two nude workers, earned his greatest success when it was featured in the New York world’s fair of 1939. Gleitsmann’s unsettling vision of industry contrasts with the favorable attitudes typically adopted by artists of a previous generation.</td>
      <td>https://clevelandart.org/art/1996.325</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=053D2DB4141F23EC9C0A4E30661B4C3963C26B53FFC42BF3CB546C5D7B7EF7AD&s=21&se=83547869&v=1&f=%5Cd9007%5Cu76090076%5C1996.325_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>317</th>
      <td>The Age of Bronze</td>
      <td>Auguste Rodin (French, 1840-1917)</td>
      <td>1875-1876</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Rodin’s earliest surviving life-size sculpture,<em> The Age of Bronze</em> is an enigmatic and provocative image of a man awakening to new consciousness. The figure originally held a spear in one hand; by removing the weapon, Rodin stripped the sculpture of narrative symbols and focused on the sensuality and psychological power of the male nude. Contemporaries found the figure so realistic they falsely accused Rodin of making a cast from a living person. Museum trustee Ralph King commissioned this cast from the artist in 1916 with the intention of donating it to the museum. Rodin personally supervised the exceptionally fine casting and finished it with his favorite patina, a deep reddish tone he called “crushed grape.”</td>
      <td>https://clevelandart.org/art/1918.328</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE8A583FA013E24A522029231A3E9F737052E2F3B641230F4F&s=21&se=1297073523&v=5&f=1918.328_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>318</th>
      <td>Figure Studies for the Sistine Ceiling (verso)</td>
      <td>Michelangelo Buonarroti (Italian, 1475-1564)</td>
      <td>1510-1511</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>The verso of this drawing features several close-up preparatory studies of the foot and toes of an <em>ignudo</em> (athletic male nude) as well as two studies of a head and shoulders that have not been linked to any figures on the ceiling of the Sistine Chapel.</td>
      <td>https://clevelandart.org/art/1940.465.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CDE735E74C0ACE20C5BB031EAC9E97719E57E26B34ACC027BF365B59E6B76955&s=21&se=1031267696&v=1&f=1940.465.b_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>319</th>
      <td>The Dutch Girl</td>
      <td>Paul Outerbridge (American, 1896-1958)</td>
      <td>1936</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Throughout his career, Outerbridge wrote numerous articles on photography, including both technical essays and more philosophical meditations on his favorite subjects—feminine beauty and photographing the nude. Eighteenth-century French paintings, particularly depictions of harem scenes, appear to have been a direct source for his erotic nudes. Both Outerbridge and the French painters he admired presented the female nude with a balance of classical, naive innocence and worldly sensuality. In this image, Outerbridge depicted a partially nude young woman, whose averted face is thrown into shadow by her lace cap. He posed the model so that the projecting ends of the cap would echo her breasts.</td>
      <td>https://clevelandart.org/art/1942.1163</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=47099380FD68FF981B4419E4FDAE822613D404CF7610BA8FA63C386ADC3948C5&s=21&se=1031267696&v=3&f=1942.1163_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>320</th>
      <td>Cut Out Nude</td>
      <td>Tom Wesselmann (American, 1931-2004)</td>
      <td>None</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1994.153</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>321</th>
      <td>Bathers Playing with a Crab</td>
      <td>Pierre-Auguste Renoir (French, 1841-1919)</td>
      <td>c. 1897</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>After viewing Renaissance paintings during a trip to Italy in 1881, Renoir attempted to bring a greater order and stability to Impressionism. He expressed this new ambition in a series of paintings of nude bathers, a subject that preoccupied him from 1883 until his death in 1919. In this painting he combined the loose, shimmering effects of pure Impressionism with the observations of solid forms in space derived from Italian Renaissance art. At the same time, the luminous colors and the long graceful curves of the backs, arms, thighs, and hair echo the soft, rounded forms of clouds and waves, suggesting a harmony between the figures and their surroundings.</td>
      <td>https://clevelandart.org/art/1939.269</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D0CD2D88DEAD73C31CBD77DD0429D157B90019EFC706B112BA54EBD8D87118B0&s=21&se=1049999129&v=&f=1939.269_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>322</th>
      <td>Reclining Nude Woman</td>
      <td>Emil Fuchs (American, 1866-1929)</td>
      <td>1926</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1926.203</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>323</th>
      <td>Fragment with Satyr and Maenad</td>
      <td>NaN</td>
      <td>300s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>This tapestry ranks among the crowning achievements of 4th-century Egyptian textiles. These sculptural figures originally stood with many others under an arcade more than 24 feet wide. Halos proclaim the special status of the graceful satyr, identified in Greek, who is dressed in a spotted skin standing beside an elegant nude maenad adorned with gold jewelry. Satyrs and maenads were followers of Dionysus, the Greek god of wine, whose cult flourished especially in Egypt among the educated of all religions. Presumably this extravagant hanging was commissioned for cultic or theatrical festivities.</td>
      <td>https://clevelandart.org/art/1975.6</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD1659218DCF98F87D5141E540475BADD37D1B8FF7AED23CCD2&s=21&se=1329612074&v=5&f=1975.6_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>324</th>
      <td>Portrait of Ib</td>
      <td>Lucien Freud (British, 1922-2011)</td>
      <td>1977-1978</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>One of the greatest realist painters of the 20th century, Lucian Freud combines traditional rendering of the nude figure with modern expressiveness of paint on canvas. Though the artist has always focused on intimate portraits of family and friends, they are rendered with meticulous and sometimes unforgiving detail. This painting depicts Freud’s daughter Isobel as a teenager. For Freud, "the paint is the person," evidenced here in his liberal use of rich impasto.</td>
      <td>https://clevelandart.org/art/1979.15</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CBCDA84370A3FFF776A40576013BF27858B4244B3FADDEEAA1&s=21&se=1329612074&v=3&f=1979.15_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>325</th>
      <td>Nude on Chief's Blanket</td>
      <td>Philip Pearlstein</td>
      <td>1978</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1979.60</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>326</th>
      <td>Andromeda</td>
      <td>Antonio Tarsia (Italian, 1663-1739)</td>
      <td>c. 1720-1730</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>To stop attacks by a sea monster sent to punish Queen Cassiopeia for bragging that she was more beautiful than the nymphs of the sea, an oracle decreed that her virgin daughter, Andromeda, be tied to a rock and sacrificed to the creature. The hero Perseus would eventually save her, but artists often chose this moment as an opportunity to display a young, nude woman, justified by a veneer of mythology.</td>
      <td>https://clevelandart.org/art/1981.227</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FF2F17B3BD48E773DC1445566FB275521E76C145FA527E4ADA&s=21&se=1329612074&v=6&f=1981.227_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>327</th>
      <td>Spoon with Saint Paul as an Athlete</td>
      <td>NaN</td>
      <td>350-400</td>
      <td>late Roman Empire</td>
      <td>None</td>
      <td>None</td>
      <td>Silver spoons with swan neck handles (ligulae) were popular in the Late Roman Empire. This late antique spoon is unique because it is decorated with the nude figure of a victorious athlete identified in an inscription as PAVLVS (Paul). It is tempting to interpret the juxtaposition of the name with a classical representation of an athlete as a subtle allusion to a passage in Saint Paul’s first letter to the Corinthians (1 Cor. 9:24-27), in which the apostle characterizes himself as an "athlete of Christ." While this interpretation may imply that the spoon’s owner was a Christian, it does not imply a religious function for the object, which was likely used for display or fine dining.</td>
      <td>https://clevelandart.org/art/1964.39</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E6BB29CD4AAB4CD10900C4737811C1E7EB8925D56B2E1D3372B016BDE5CA28F5&s=21&se=1279503776&v=8&f=1964.39_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>328</th>
      <td>Head of a Boy</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1905-1906</td>
      <td>Spain</td>
      <td>Created during Pablo Picasso's Rose Period (1904–6), this drawing features a warm palette and thin delicate lines. It might have been created during a summer that Picasso spent in Gósol, a Catalan village in the Spanish Pyrenees. The sheet presents an adolescent with pensive lips, almond-shaped eyes, and a dreamy expression—all characteristics that recur throughout the artist's depictions of figures around this time. Picasso began with a layer of pink opaque paint, allowing the color to give the boy's skin a warm tone that contrasts with the gray used to delineate his head. The subject's skull and jaw were then carefully modeled using a darker tone in roughly applied marks. The close-up view—stark in the present drawing—was later reused in Picasso's 1906 painting <em>The Two Brothers</em> (Kunstmuseum Basel), in which a nude adolescent carries a young boy on his back.</td>
      <td>This Picasso drawing was once owned by the American writer Gertrude Stein (1874–1946); a photograph of her apartment in Paris taken around 1907 shows the work hanging there.</td>
      <td>The delicate facial features, pensive lips, and almond-shaped eyes of this dreamy adolescent are characteristic of similar figures that appear repeatedly in Picasso's paintings of harlequin and circus figures of 1905. This work was owned by the American writer and patron of the artist, Gertrude Stein (1874-1946), and a photograph of her apartment in Paris taken around 1907 shows the work hanging there.</td>
      <td>https://clevelandart.org/art/1958.43</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504B12E38491D8C276E05EC0AC1B73CE57DC895DD8BCD27DF072&s=21&se=995010993&v=3&f=1958.43_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>329</th>
      <td>Mountain Climber</td>
      <td>Rockwell Kent (American, 1882-1971)</td>
      <td>1932-1933</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Kent was a wanderer and adventurer, who traveled to Alaska, Patagonia, and Greenland. Much of his work, like this drawing, celebrates heroic, manly figures. Interestingly, Kent did not use professional models but drew from his own nude body with the help of mirrors.</td>
      <td>https://clevelandart.org/art/1933.143</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57CCC07FCBB9C197173422015F54DE7BC47E15F9AEC7886AAF&s=21&se=1049999129&v=2&f=1933.143_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>330</th>
      <td>Ornamental Shoulder Bands from a Tunic</td>
      <td>NaN</td>
      <td>500s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>The orientation of the figures indicates that these were vertical bands on a tunic descending from the shoulders. Human figures under arches alternate with dogs, rabbits, and a lion. The right band displays a nude female dancer playing finger cymbals at the top and bottom, and in the center, a hunter dressed in a short garment (chiton) draped over one shoulder carries a shield in his right hand and a rock in his raised hand.\r\n\r\nThe left band displays, from top to bottom, a nude male carrying a leafy branch; a shepherd carrying a shepherd’s crook and dressed in a skirt of animal skin; and a nude female dancer with a scarf loosely draped behind her.</td>
      <td>https://clevelandart.org/art/1926.145.b</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=FDF737B1B5E2E5DECB8BA2A847C7942DB1576DB78F598180BE82470FCBA6D75D&s=21&se=1049999129&v=&f=1926.145.b_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>331</th>
      <td>Aphrodite Torso</td>
      <td>NaN</td>
      <td>2nd-1st Century BC</td>
      <td>Greece</td>
      <td>None</td>
      <td>This torso is based on the Aphrodite of Knidos, a Greek original by the master sculptor Praxiteles.</td>
      <td>No single sculpture in the history of Western art has been more influential than Praxiteles’s Knidian Aphrodite, carved in the 4th century BC. The first sculpted nude figure of the goddess of love, it became a famous tourist attraction and made Praxiteles a celebrity in the Greek world. The shocking originality of showing the goddess unaware that she had been seen emerging from her bath made a lasting impression. It became a prototype for generations of Greek and Roman sculptors. Its popularity inspired many variations, including this one. The sensitivity of the carving, seen to advantage in the wet hair falling on the shoulders, imparts a dynamic sensuality.</td>
      <td>https://clevelandart.org/art/1988.9</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=DA3BCC800F2227E229790C8BFC51DBF071128905A941CE895CF63119C615C97C&s=21&se=1141978221&v=1&f=%5Cd4629%5Cu120346290%5C1988.9_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>332</th>
      <td>Last Judgement (Selfless, Deathless, No World)</td>
      <td>Greg Parma Smith (American, b. 1983)</td>
      <td>2016</td>
      <td>Greece</td>
      <td>The painting’s title references the final day of reckoning in the Christian tradition when all of humanity stands before God for final judgement. In this six-panel painting, the artist uses a multitude of cultural and religious symbols surrounding a rising or setting sun. The most striking is a nude woman striding forward, perhaps referencing a Christ figure. The rolled sections reveal diverse layered imagery, further suggesting the complexities of contemporary life.</td>
      <td>This artist is known for his precise painterly realism as seen in this work.</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2016.299</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C4FDB92EAE3E92E83D1AC82BF4E37BE3F1CD925AA6A9A949BCDFBBB2F7629038&s=21&se=676011667&v=1&f=%5Cd9927%5Cu52699270%5C2016.299_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>333</th>
      <td>Solidarity</td>
      <td>George Minne (Belgian, 1866-1941)</td>
      <td>1898</td>
      <td>Belgium</td>
      <td>None</td>
      <td>None</td>
      <td>In 1898 the Belgian Labor Party commissioned from Minne a memorial for the Socialist leader Jean Volders. The sculptor decided that an allegory of brotherhood-consisting of two nude youths standing in a boat, anxiously trying to keep their balance in a storm-would be more appropriate than a traditional portrait of the deceased. The Labor Party disliked the design and refused to have it executed in stone. In anger, Minne destroyed his large plaster model, but a smaller study remained in his studio and was later cast in bronze. This marble version is the only carved example of the composition known to have been completed.</td>
      <td>https://clevelandart.org/art/1968.191</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F8BC7E8397D9E664B364D20A9B04E1ABADCC68D636CCC2882C73B3532560F268&s=21&se=172172604&v=5&f=1968.191_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>334</th>
      <td>Torso of a Kouros</td>
      <td>NaN</td>
      <td>575–550 BC</td>
      <td>Greece</td>
      <td>None</td>
      <td>None</td>
      <td>Known as a <em>kouros,</em> the ancient Greek word for a youth, this fragmentary statue belongs to a relatively rare type of large-scale stone sculpture made for little more than a century—shortly before 600 BC to soon after 500 BC. A nude youth standing with arms to the sides and one foot slightly advanced, the type probably originated under Egyptian influence, but then developed along entirely Greek lines. Found in sanctuaries as well as cemeteries across the ancient Greek world, <em>kouroi </em>may represent gods—especially Apollo—as well as mortals. Although incomplete, this kourosretains much of its beautifully finished and patterned surface.</td>
      <td>https://clevelandart.org/art/1953.125</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CB670FEC2161B3E7DB1F47983AE3360250E6F2E62186C48B298CA3A2BA7BF811&s=21&se=1031267696&v=5&f=1953.125_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>335</th>
      <td>Portrait of Napoléone Elisa Baciocchi, Niece of Napoleon I</td>
      <td>Lorenzo Bartolini (Italian, 1777-1850)</td>
      <td>1810-1812</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>To extend power across the continent, Napoleon arranged for his sisters to marry into the courts of Europe. The sitter is his niece, daughter of the Grand-Duke of Tuscany (the bee on her cup is a Napoleonic emblem). While the girl’s nakedness might startle us today, in the early 1800s depicting children nude emphasized their purity and innocence. The work takes its cues from ancient sculpture, and while the pet dog adds a note of tenderness, it also refers to Diana, goddess of the moon and hunt.</td>
      <td>https://clevelandart.org/art/1979.37</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE19BA4AA40DC61DB6685B20A12FE2F2148DD6C1B55E3214E1&s=21&se=1329612074&v=5&f=1979.37_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>336</th>
      <td>Studies of a Seated Female, Child's Head, and Three Studies of a Baby</td>
      <td>Raphael (Italian, 1483-1520)</td>
      <td>c. 1507-1508</td>
      <td>Italy</td>
      <td>This drawing is from Raphael’s "pink sketchbook," comprised of ten sheets of roughly equal size that each portray variations of a mother and child. Today, six of the drawings at the Palais des Beaux-Arts, Lille; two are at the British Museum; one is in a private collection; and one is in Cleveland. The small format of the sheets would have enabled the artist to carry the notebook as he traveled from Florence to Rome in 1508. Raphael used metalpoint, a technique popular in 15th- and early 16th-century Italy on a pink prepared surface. The pose of the infant's head in the drawing was based on that of the Christ child in Leonardo da Vinci's <em>Benois Madonna</em>, then in a private collection in Florence, with changes—such as the uplifted eyes and open mouth—made by Raphael. The curving back of the female nude is echoed in the roundness of the child's head. The three sketches of a reclining infant at the bottom of the sheet are freely handled and improvisational, relaying the child's squirming, continuous movement with repeated contour lines.</td>
      <td>This sheet was probably once part of a sketchbook carried by the artist Raphael on a 1508 journey between Florence and Rome.</td>
      <td>This drawing is from Raphael’s "pink sketchbook," comprised of ten sheets of roughly equal size portraying a mother and child. Today, six are at the Palais des Beaux-Arts, Lille; two are at the British Museum; one is in a private collection; and one is in Cleveland. The small format of the sheets would have enabled the artist to carry the notebook as he traveled from Florence to Rome in 1508. Raphael used metalpoint, a technique popular in 15th- and early 16th-century Italy. As the name implies, metalpoint employed a stylus of soft metal—usually silver or a silver alloy—that left a mark when applied to a specially prepared sheet coated with ground eggshell, powdered bone, or lead white. The rough texture allowed for a deposit of the metal to be left on the surface. The preparation of the ground was often tinted with pigment (here Raphael used pink) so that the delicate gray lines of the metalpoint would stand out. The rigorous technique cannot be erased or effaced, thus demonstrating the artist’s superlative draftsmanship.</td>
      <td>https://clevelandart.org/art/1978.37</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F93CB46F71431BEEF187CA8EF077E037AC318343B2386FC2C2671342026A92FB&s=21&se=1523204814&v=4&f=1978.37_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>337</th>
      <td>Decorations and Sleeve from a Tunic</td>
      <td>NaN</td>
      <td>700s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>Classical figures and winged animals remained popular during the early Islamic period. The nude male may portray Dionysus, Greek god of wine; he holds his thyrsus-a staff ornamented with ivy leaves and pine cones-as he pours liquid from a small jug for the panther. These colorful designs would have decorated the front or back of a tunic and one sleeve.\r\n\r\nElaborate embellishments were cherished. Thus, once a tunic began to fray, its decoration was sewn onto a new one, as seen on this winterweight woolen tunic cloth. Its finely detailed motifs, some with eccentric drawing, are woven in tapestry weave-the equivalent of painting with weft thread; discontinuous horizontal wefts are interlaced only where needed in the design.\r\n\r\n</td>
      <td>https://clevelandart.org/art/1982.107</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D5DDA196E97E8D6E40E9C9A1E4D11F498393DE40D9B567C469B4666DCB2F6248&s=21&se=1329612074&v=&f=1982.107_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>338</th>
      <td>The Hamadryads</td>
      <td>Anne W. Brigman (American, 1869-1950)</td>
      <td>c. 1910</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>During the first two decades of the 20th century, Anne Brigman was a leading practitioner of the pictorial photography style. She often used mythological themes rendered in soft focus, and manipulated her prints and negatives to remove her photographic subject matter from common, everyday reality. This print exemplifies Brigman's allegorical photography, in which nude figures were integrated into natural settings. In Greek mythology a Hamadryad is a nymph whose life begins and ends with that of a specific tree. Here, two nude females, representing "wood nymphs," were carefully placed among the flowing forms of an isolated tree in the Sierra Nevada mountains, a favorite location for much of Brigman's work.</td>
      <td>https://clevelandart.org/art/1995.77</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762B6F609DF5E7437650BD35B0F7C1DB85691B2B5C4D7070EF25&s=21&se=686303180&v=3&f=1995.77_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>339</th>
      <td>Ornamental Shoulder Bands from a Tunic</td>
      <td>NaN</td>
      <td>500s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>The orientation of the figures indicates that these were vertical bands on a tunic descending from the shoulders. Human figures under arches alternate with dogs, rabbits, and a lion. The right band displays a nude female dancer playing finger cymbals at the top and bottom, and in the center, a hunter dressed in a short garment (chiton) draped over one shoulder carries a shield in his right hand and a rock in his raised hand.\r\n\r\nThe left band displays, from top to bottom, a nude male carrying a leafy branch; a shepherd carrying a shepherd’s crook and dressed in a skirt of animal skin; and a nude female dancer with a scarf loosely draped behind her.</td>
      <td>https://clevelandart.org/art/1926.145</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EAD82D6C274EE15952E19D60687AD273EE44F4A3B8A1053E0C1E298F4C07B1C4&s=21&se=1049999129&v=&f=1926.145_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>340</th>
      <td>Sleeve from a Tunic</td>
      <td>NaN</td>
      <td>700s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>Classical figures and winged animals continued to be popular during the early Islamic period. The nude male may depict Dionysus (Greek god of wine) holding his thyrsos staff, ornamented with ivy leaves and pine cones, and pouring liquid from a small jug for the panther. The colorful and detailed winter-weight woolen tunic fragments would have decorated a tunic as seen in the square displayed nearby showing two figures within a jeweled border (1979.58).\r\n\r\nPhoto of:\r\nDetail of the Panther's Head, Right Decorative Band from the Front or Back of a Tunic (1982.107.b)\r\n\r\nThe facial features were achieved in tapestry weave by interlacing colored horizontal wefts only where they were needed in the panther's head.\r\n</td>
      <td>https://clevelandart.org/art/1982.107.c</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D5DDA196E97E8D6EA93D0D7A7C7F9040EF0FFAEED51DC1D1A53DA96C18F2A9C7&s=21&se=1329612074&v=&f=1982.107.c_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>341</th>
      <td>The Four Festivals:  Festival of Diana</td>
      <td>Claude Gillot (French, 1673-1722)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>This depiction of the festival of Diana, Roman goddess of forests and animals, accompanies three other scenes glorifying the nature gods Faunus, Bacchus, and Pan. In each print a frame of flourishing vegetation surrounds nude and semi-nude figures who frolic around an altar with a bust to the god. The caption below the Festival of Diana declares that this celebration is being "troubled by satyrs," whose muscular, goat-legged bodies and leering faces appear at either edge of the \r\nwooded grove.\r\n</td>
      <td>https://clevelandart.org/art/1985.97.1</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=40675183989CEBD83107D2505E443436FE259B0603D57A61D3E94B12C82C6BE7&s=21&se=951169631&v=5&f=1985.97.1_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>342</th>
      <td>Le Cirque de l'étoile filante:  Nude</td>
      <td>Georges Rouault (French, 1871-1958)</td>
      <td>1938</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1956.291</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>343</th>
      <td>The Four Festivals</td>
      <td>Claude Gillot (French, 1673-1722)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>This depiction of the festival of Diana, Roman goddess of forests and animals, accompanies three other scenes glorifying the nature gods Faunus, Bacchus, and Pan. In each print a frame of flourishing vegetation surrounds nude and semi-nude figures who frolic around an altar with a bust to the god. The caption below the Festival of Diana declares that this celebration is being "troubled by satyrs," whose muscular, goat-legged bodies and leering faces appear at either edge of the \r\nwooded grove.\r\n</td>
      <td>https://clevelandart.org/art/1985.97</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F88318034C64C677EB3F801718A9579B1FB1A9DE10FF5F2D19EDD38B32EF73EA&s=21&se=166975769&v=&f=1985.97_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>344</th>
      <td>Clavi (Decorative Band)</td>
      <td>NaN</td>
      <td>700s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>Classical figures and winged animals remained popular during the early Islamic period. The nude male may portray Dionysus, Greek god of wine; he holds his thyrsus-a staff ornamented with ivy leaves and pine cones-as he pours liquid from a small jug for the panther. These colorful designs would have decorated the front or back of a tunic and one sleeve.\r\n\r\nElaborate embellishments were cherished. Thus, once a tunic began to fray, its decoration was sewn onto a new one, as seen on this winterweight woolen tunic cloth. Its finely detailed motifs, some with eccentric drawing, are woven in tapestry weave-the equivalent of painting with weft thread; discontinuous horizontal wefts are interlaced only where needed in the design.\r\n\r\n</td>
      <td>https://clevelandart.org/art/1982.107.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D5DDA196E97E8D6E5CCF026B6B19E3609B67162BC36F6E8F4847F4A9B44CDC95&s=21&se=1329612074&v=&f=1982.107.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>345</th>
      <td>Ornamental Shoulder Bands from a Tunic</td>
      <td>NaN</td>
      <td>500s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>The orientation of the figures indicates that these were vertical bands on a tunic descending from the shoulders. Human figures under arches alternate with dogs, rabbits, and a lion. The right band displays a nude female dancer playing finger cymbals at the top and bottom, and in the center, a hunter dressed in a short garment (chiton) draped over one shoulder carries a shield in his right hand and a rock in his raised hand.\r\n\r\nThe left band displays, from top to bottom, a nude male carrying a leafy branch; a shepherd carrying a shepherd’s crook and dressed in a skirt of animal skin; and a nude female dancer with a scarf loosely draped behind her.</td>
      <td>https://clevelandart.org/art/1926.145.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5D0F103AF8C2334297D1DA0D002107FBD1DDCFEC3D65E7C70A932575B2359D5C&s=21&se=1049999129&v=&f=1926.145.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>346</th>
      <td>Devant le miroir</td>
      <td>Edouard Vuillard (French, 1868-1940)</td>
      <td>c. 1898</td>
      <td>Egypt</td>
      <td>During the early 1890s, Édouard Vuillard repeatedly represented his mother and older sister, Marie, in domestic settings. This drawing relates to a pastel and painting, all of which show Marie standing before a mirror, raising her arms as if to adjust her hairstyle. A charcoal study of a nude male appears on the verso, suggesting that the artist may have reused the sheet.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.77</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=81064D36B0DB749252D0B87E657EF21B27B8B2E65BB7C6FAD1886B66530557DD&s=21&se=402160102&v=1&f=2018.77_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>347</th>
      <td>Architectural Bracket with Wrestlers</td>
      <td>NaN</td>
      <td>100s-200s</td>
      <td>Pakistan</td>
      <td>This sculpture probably derived from Greco-Roman prototypes of nude athletes, but here it has probably been transposed into a scene from the boyhood of the Buddha, when it is said that he excelled in all forms of athletics.</td>
      <td>None</td>
      <td>This sculpture probably derived from Greco-Roman prototypes of nude athletes, but here it has probably been transposed into a scene from the boyhood of the Buddha, when it is said that he excelled in all forms of athletics.</td>
      <td>https://clevelandart.org/art/1998.71</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=BDFDD69D477608B19AD3A345538C3AC7B8338A930E4AA3F73BDC719E8224693D&s=21&se=686303180&v=4&f=1998.71_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>348</th>
      <td>The Sirens</td>
      <td>Auguste Rodin (French, 1840-1917)</td>
      <td>1889</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Rodin intended <em>The Gates of Hell</em> (based on Dante’s <em>The Divine Comedy</em>) to be the great work of his life. It was never executed as originally planned, but Rodin created a number of sculptures for the project that were later treated separately. Among them was this group of three nude women entitled <em>The Sirens</em>.</td>
      <td>https://clevelandart.org/art/1946.350</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504B5E301AC158ED3E9AEE35B99269E543B3FA7AF63F8B73125E&s=21&se=1031267696&v=6&f=1946.350_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>349</th>
      <td>In the Studio</td>
      <td>Lovis Corinth (German, 1858-1925)</td>
      <td>1919</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>Made in the final years of Lovis Corinth’s life, this self-portrait shows him standing in his studio with his wife, Charlotte, posed behind him in the nude with shaded features. Corinth had suffered a stroke in 1911, but with the help of Charlotte he continued to make art focusing largely on his family life. Charlotte’s presence indicates her importance to him as both supporter and muse.</td>
      <td>https://clevelandart.org/art/1966.412</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=92B37231D833A78085CB979F3D996BFE67812CC2BD5D885F629387023A2A2B92&s=21&se=172172604&v=4&f=1966.412_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>350</th>
      <td>The Lover Surprised (L'Amant Surpris)</td>
      <td>Frédéric Schall (French, 1752-1825)</td>
      <td>c. 1798</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>A young woman surprises her sweetheart, whom she has found reading love letters in a wooded park. The love-struck fellow carries a miniature portrait of her as a keepsake. Verdant foliage, a nearby brook, and a blooming rosebush signal fertility and the bounty of life. The nude female statue sitting on a pedestal seems to watch the playful lovers.</td>
      <td>https://clevelandart.org/art/2015.151</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=49150E92B341196C3C36CEED8E3BC5AB09B7D7B9EA3A9AF362045C5042B8C00A&s=21&se=1789656799&v=&f=2015.151_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>351</th>
      <td>Women with Parasols; Walking Nude; Reclining Nude, No. 1; and Woman Combing Her Hair</td>
      <td>Charles Frederick Ramus (American, 1902-1979)</td>
      <td>1929</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1930.102</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>352</th>
      <td>Diary Nov. 5th '85</td>
      <td>Tetsuya Noda (Japanese, 1940-)</td>
      <td>1985</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.174</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>353</th>
      <td>Standing Figurine</td>
      <td>NaN</td>
      <td>1200-900 BC</td>
      <td>Central Mexico</td>
      <td>None</td>
      <td>None</td>
      <td>Prior to the Olmec, clay figurines, mainly nude females, were the most common art of Mesoamerica. The meanings of these small, intimate works are not known, but they are found both in burials and trash, suggesting varied uses. These two figurines are in the Tlatilco style of central Mexico; they represent the artistic context in which the Olmec style developed.</td>
      <td>https://clevelandart.org/art/1962.250</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EB63BA0A504267D78DC47B0E28F16147DDB0BA5CFAA642273DB399C100C19005&s=21&se=1674154316&v=3&f=1962.250_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>354</th>
      <td>Male Academy with Wings</td>
      <td>François Boucher (French, 1703-1770)</td>
      <td>1745-1750</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Academically trained artists often practiced drawing live models. Called academies, these figure studies occasionally served as preparatory drawings for paintings. Some artists crafted their academies to appeal to collectors interested in contemplating technical skill and inventiveness. Boucher may have imaginatively given wings to this male nude, who does not appear in any known painting, to demonstrate his ingenuity by transforming the model into a mythological figure.</td>
      <td>https://clevelandart.org/art/2008.350</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B69E2CB8AE858722B13A306B453E4A49FC527132E56A7E44740FFBA6F349EC41&s=21&se=1579770806&v=2&f=2008.350_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>355</th>
      <td>Study of Two Soldiers</td>
      <td>Théodore Géricault (French, 1791-1824)</td>
      <td>1818-1819</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2008.371</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B69E2CB8AE858722A531380B5D58D7CEE35B4D9C70D77BF14F0C7BA92F38F81F&s=21&se=1285208684&v=2&f=2008.371_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>356</th>
      <td>Study for the Mother in The Fisherman's Family</td>
      <td>Pierre Puvis de Chavannes (French, 1824-1898)</td>
      <td>c. 1875</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>This study relates to the figure of the mother in <em>The Fisherman's Family</em>, a painting first shown in the 1875 Salon exhibition, and which the artist also painted in reduced scale in 1887. While Puvis represented his female figure completely nude in the study, in the painting he left bare just a breast, shoulder, and arm.</td>
      <td>https://clevelandart.org/art/2008.390</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B69E2CB8AE858722F57EFBA588FE779FD0786F52EC24489C7DD9C2DB38351ACE&s=21&se=1579770806&v=2&f=2008.390_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>357</th>
      <td>The Coiffure</td>
      <td>Mary Cassatt (American, 1844-1926)</td>
      <td>1890-1891</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>The theme of a woman at her toilette was a common one for male artists of the 19th century for whom it presented the opportunity to depict the female nude—historically considered an essential aspect of artistic practice. For a woman artist to address the subject transgressed social norms of the period. Women were not permitted to attend the illustrious École des Beaux-Arts until 1897, and even at the Académie Julian, which opened to women in 1868, they were not permitted to draw from nude models. Cassatt's answer to the rules of decorum was to suggest the setting of a bourgeois house, rather than the settings her male contemporaries preferred—such as the Turkish bath or the brothel. In her description of the female body, Cassatt relied upon the stylized elegance of her Japanese prototypes, thereby abstracting the nudity of her model.</td>
      <td>https://clevelandart.org/art/1941.79</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F5DB0801F450918FD1F7690550BF8077225BA10C53D9B4B15728DF5D39C6303C&s=21&se=1031267696&v=3&f=1941.79_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>358</th>
      <td>Pomona</td>
      <td>Andrea Riccio (Italian, c. 1470-1532)</td>
      <td>c. 1500</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>According to Roman mythology, the nymph Pomona cared for the gardens and forests, rejecting the pursuits of her suitors. The god Vertumnus fell in love with her. He disguised himself as an old woman, complimented her orchard and kissed Pomona passionately. As an old woman, Vertumnus explained that there was one person who loved both Pomona and her garden. Vertumnus then revealed himself to her, and from then on the two cared for Pomona's gardens together. Classicizing in style, Pomona stands nude with her weight on her left leg. She holds a bowl of fruit in her left hand while her right hand is clenched, suggesting it may have once held another object. Pomona wears a diadem, or headband, and hair decorated with a beaded garland falls in two locks over her shoulders. A hole has been drilled at the top of the figure's head for removal of the core and casting pins appear in the back of the right shoulder and on the front of the left leg. Other examples of this statuette exist in Oxford, Paris, Berlin and Washington. Only the Washington and Cleveland versions have been attributed to Riccio because of their superior quality of casting and the rich patinas.</td>
      <td>https://clevelandart.org/art/1948.486</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57AB0B69EC7339FAA6F8CF58A8452EEF3DAF422BB9D925E800&s=21&se=1031267696&v=5&f=1948.486_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>359</th>
      <td>Pax with the Adoration of the Magi</td>
      <td>Moderno (Italian, 1467-1528)</td>
      <td>c. 1500</td>
      <td>Italy</td>
      <td>A pax is a small flat tablet decorated with Christian images and meant to be kissed by the priest before Communion.  This plaquette likely had a tabernacle-shaped frame which has been cut away.  Here the Madonna sits before a stable with a nude Christ Child seated on her right knee.  A star with a long vertical ray radiates in the distance above the stable.  Joseph stands on the right edge of the relief, while a nude child, likely a young John the Baptist, appears beside Joseph.  The Three Kings approach the Holy Family with gifts, the first kneeling, his arms folded across his chest, while a kneeling male, nude except for a helmet, adjusts the sandal of the third king.  A long procession follows the magi, continuing diagonally up the rocky hills in the background.  This rocky landscape and crowded figures animate the surface, conveying the excitement and haste as visitors journey to see the newborn king.  \r\n\tAn inscription on the back of the plaquette reads "This has been given as a gift to Andrea Aiardo, Archbishop of Brundisi by Ioannes Baptista Fontanus, Goldsmith, in the year of our Lord 1583."  Other versions of this plaquette exist in the British Museum, the Victoria and Albert Museum, the Louvre, the Berlin State Museums, and the Metropolitan Museum of Art.  \r\n\r\n--Ana Dieglio (2009)</td>
      <td>None</td>
      <td>A pax is a small flat tablet decorated with Christian images and meant to be kissed by the priest before Communion.  This plaquette likely had a tabernacle-shaped frame which has been cut away.  Here the Madonna sits before a stable with a nude Christ Child seated on her right knee.  A star with a long vertical ray radiates in the distance above the stable.  Joseph stands on the right edge of the relief, while a nude child, likely a young John the Baptist, appears beside Joseph.  The Three Kings approach the Holy Family with gifts, the first kneeling, his arms folded across his chest, while a kneeling male, nude except for a helmet, adjusts the sandal of the third king.  A long procession follows the magi, continuing diagonally up the rocky hills in the background.  This rocky landscape and crowded figures animate the surface, conveying the excitement and haste as visitors journey to see the newborn king.  \r\n\tAn inscription on the back of the plaquette reads "This has been given as a gift to Andrea Aiardo, Archbishop of Brundisi by Ioannes Baptista Fontanus, Goldsmith, in the year of our Lord 1583."  Other versions of this plaquette exist in the British Museum, the Victoria and Albert Museum, the Louvre, the Staatliche Museum in Berlin, and the Metropolitan Museum of Art.  \r\n</td>
      <td>https://clevelandart.org/art/1963.254</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE94DB86AB398BF79BC2D43986962BEADF942E8DB61EE57F14&s=21&se=418730041&v=5&f=1963.254_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>360</th>
      <td>François Tronchin</td>
      <td>Jean-Etienne Liotard (Swiss, 1702-1789)</td>
      <td>1757</td>
      <td>Switzerland</td>
      <td>None</td>
      <td>None</td>
      <td>This magnificent pastel depicts François Tronchin, a prominent figure in his native Geneva and an impassioned patron of the arts. On the table are a book, mathematical instruments, and papers that indicate his taste for architecture and music. Rembrandt's Lady in Bed, about 1645-46 (Scottish National Gallery), the most highly prized painting in Tronchin's collection, rests on an easel. Liotard considered the portrait of Tronchin among his finest works. Indeed, the meticulous rendering of the sitter's powdered wig, transparent flesh, and lace cuffs, accompanied by the conjuring of the subtle grace of the Dutch master's nude, attest to the pastelist's consummate skill.</td>
      <td>https://clevelandart.org/art/1978.54</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1EF2524F1C2C504BFB5A52BCE248112C779C46F2F106E225A38967EFD61ADE78&s=21&se=1329612074&v=&f=1978.54_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>361</th>
      <td>Paris: The Eiffel Tower</td>
      <td>Joseph Hecht (French, 1891-1951)</td>
      <td>1933</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2003.324</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=6A111FB3078589B7945E5E2B76C878C6D16784B5A179C398CE129FAD364BF330&s=21&se=1755401958&v=6&f=2003.324_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>362</th>
      <td>The Game of Blind Man's Bluff</td>
      <td>Nicolas Lancret (French, 1690-1743)</td>
      <td>1739</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Specially trained printmakers like Cochin reproduced and popularized other artists’ works in printed form. His free and spontaneous style of etching translates Lancret’s original painting with a lighthearted quality appropriate to this bucolic setting and <em>fête champêtre,</em> a festive outdoor party enjoyed especially by members of the French aristocracy. The fashionable garden’s rococo balustrade, with its cascading arrangement of nude statues, is reminiscent of Boucher’s fountain designs on view nearby.</td>
      <td>https://clevelandart.org/art/1974.94</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=33B458E6429A5D59DF997E9CE967AB88BF17A933D9C342A5BCB01965F32170CB&s=21&se=1065507836&v=&f=1974.94_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>363</th>
      <td>Woman Seen from the Back</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>None</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Maillol was interested in conveying the sensual curves of the female form. As a sculptor he worked from both memory and drawings. He made numerous studies of the nude which are related to, if not directly preparatory for his sculptures. He explained, "The important thing is the general idea...I am seeking beauty, not character."</td>
      <td>https://clevelandart.org/art/1941.504</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E6803453F000FA82EE374D8506E73F9BEBDCE54F2AACA50D9&s=21&se=1031267696&v=3&f=1941.504_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>364</th>
      <td>Standing Figurine</td>
      <td>NaN</td>
      <td>1200-900 BC</td>
      <td>Central Mexico</td>
      <td>None</td>
      <td>None</td>
      <td>Prior to the Olmec, clay figurines, mainly nude females, were the most common art of Mesoamerica. The meanings of these small, intimate works are not known, but they are found both in burials and trash, suggesting varied uses. These two figurines are in the Tlatilco style of central Mexico; they represent the artistic context in which the Olmec style developed.</td>
      <td>https://clevelandart.org/art/1952.455</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EB63BA0A504267D7A33221D171EF38E09262A68CF8966E1E6D646FC808B740A4&s=21&se=1031267696&v=3&f=1952.455_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>365</th>
      <td>Devant le miroir (recto)</td>
      <td>Edouard Vuillard (French, 1868-1940)</td>
      <td>c. 1898</td>
      <td>France</td>
      <td>During the early 1890s, Édouard Vuillard repeatedly represented his mother and older sister, Marie, in domestic settings. This drawing relates to a pastel and painting, all of which show Marie standing before a mirror, raising her arms as if to adjust her hairstyle. A charcoal study of a nude male appears on the verso, suggesting that the artist may have reused the sheet.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.77.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=81064D36B0DB7492E73BA36A0330AB7CEE4D33F0CB7D9FFF0CFE4895328AB97E&s=21&se=402160102&v=1&f=2018.77.a_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>366</th>
      <td>The Sea Monster</td>
      <td>Albrecht Dürer (German, 1471-1528)</td>
      <td>c. 1501</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>The precise narrative of Dürer’s Sea Monster remains a source of debate among scholars because locating the origin of this imagery in either classical or German mythology has been difficult. The engraving depicts a woman’s abduction by a horned mythical hybrid creature that has the torso of a man and the tail of a fish. Set before a detailed coastal landscape featuring Nuremberg castle, the woman’s companions across the river flail their arms in distress over her kidnapping. While it is clear that Dürer aimed to showcase his achievements in portraying a reclining female nude, her somewhat blasé appearance and lack of struggle add to the peculiarity of this image.</td>
      <td>https://clevelandart.org/art/1934.340</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA720797E0DB854C4AAD8F775246A84FE375FEB860DCDCA7C17A&s=21&se=1049999129&v=3&f=1934.340_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>367</th>
      <td>Femme accoudée, sculpture de dos et tête barbue, from the Vollard Suite</td>
      <td>Pablo Picasso (Spanish, 1881-1973)</td>
      <td>1933</td>
      <td>Germany</td>
      <td>This print belongs to a series of 100 that the dealer Ambroise Vollard commissioned from Picasso during the 1930s. Picasso worked with master printer Roger Lacourière, experimenting with a range of etching techniques. Many of the prints, such as this one, feature a stark, linear style evocative of classical art that Picasso favored during this period. Here, a nude sculpture and heavily bearded face further suggest ancient Greece.</td>
      <td>Although Ambroise Vollard commissioned this print from Picasso in 1930, it was not printed until the 1939, the year of Vollard’s death.</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.1078</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=736D0677D9A139D6F73E7F1BB9FF075317AEE7894899E8B9592BDB57E52A6668&s=21&se=402160102&v=1&f=2018.1078_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>368</th>
      <td>Vénus Astarté (Semitic goddess of fertility and love)</td>
      <td>Auguste Rodin (French, 1840-1917)</td>
      <td>c. 1900</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>This relief of a female nude standing in water may represent the only time the great French sculptor Auguste Rodin provided an ornament for a precious object: a gold and enamel hand mirror (see photo) designed by Felix Braquemond (1833-1914). The mirror-acquired by the museum in 1978-was made for Baron Joseph Vitta, one of Rodin's most important patrons during the years around 1900, and it was probably for this reason that the artist agreed to the commission. This plaster cast of Rodin's nude was an intermediate version of the subject. Although the images on the mirror and the plaster cast were clearly derived from the same mold, there are significant differences in the figure's hair. Rodin probably changed the wax version (from which the gold relief of the mirror was made by lost-wax casting) before he used it.</td>
      <td>https://clevelandart.org/art/2000.22</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=AB88F68620E712633CAED395D0F3E67E4FA829522D0DE2583A10548919846B4C&s=21&se=1755401958&v=7&f=2000.22_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>369</th>
      <td>Venus Wounded by a Rose's Thorn</td>
      <td>Antonio Salamanca (c.1500-1562)</td>
      <td>c. 1516</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>This composition alludes to <em>The Lament for Adonis</em> by the Greek poet Bion (active about 100 BC). In the poem, Venus, distraught by the death of her lover Adonis, wanders barefoot in the woods and is wounded by brambles. Although Bion implores Venus to “weep no longer in the thickets,” the poem does not describe the moment depicted here when she plucks a thorn from her foot, imaginatively conceived as a vehicle to present a classical female nude. The wide-eyed hare near Venus is an ancient symbol of fertility and sexual desire.</td>
      <td>https://clevelandart.org/art/1930.581</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=759357B86A0AD4F4042E9F54DF399966BA38DE2243348A6C6F1664F8EDCA4493&s=21&se=1049999129&v=&f=1930.581_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>370</th>
      <td>The Temptation of St. Anthony</td>
      <td>Julien Léopold Boilly (French, 1796-1874)</td>
      <td>19th century</td>
      <td>France</td>
      <td>Boilly’s scene represents the Temptations of Saint Anthony, a biblical subject. While embarking on his desert pilgrimage through Egypt, Anthony experienced a series of supernatural trials. Here, Boilly illustrated a nude nymph attempting to seduce him. Anthony’s horrified reaction to the woman’s embrace and the prominent crucifix on the right create a humorous juxtaposition between piety and lustful desire.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/2018.42</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=3AEF62CDCF4532159A3E7E71838C563A6E2A2F7C771E91CB8ABAA564764C08C7&s=21&se=2006672921&v=1&f=2018.42_w.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>371</th>
      <td>Lion Hunt</td>
      <td>Moderno (Italian, 1467-1528)</td>
      <td>1581 or later</td>
      <td>Netherlands or Germany</td>
      <td>The front of this medal contains an animated lion hunt; a nude male defends himself with his shield against an attacking lion while other nude horsemen and soldiers join in the fray. Contrasted against this high relief and classical motif, the reverse has an engraving of ten scenes from the Passion of Christ organized within cloister-like architecture. Starting with the "Last Supper" (on the far left in the center row), the story moves clockwise, ending in the two central scenes that depict Christ's deposition in the tomb and then his miraculous resurrection. The date 1581 is engraved on the altar in the scene of the resurrected Christ. The same Passion composition was also used for the reverse of a 1564 medal for the King of Poland, Sigismund Augustus (1520-1572). The tooling of the relief and the format of the Passion scenes correspond to prototypes and workmanship that suggest the medal was made by a Northern follower of Moderno.</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1968.26</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE2138C8654FE024C095C13AB24E375F9182EC4FA33671738B&s=21&se=23404858&v=4&f=1968.26_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>372</th>
      <td>Female Figurine</td>
      <td>NaN</td>
      <td>c. 1200-900 BC</td>
      <td>Central Mexico</td>
      <td>This ceramic figurine—in the style of Tlatilco, an early village site near Mexico City—depicts a nude female with an elaborate coiffure, attenuated arms, and the traces of mineral pigment. Since many figurines from the period depict females, modern interpreters usually connect them to fertility concerns. At Tlatilco, figurines were found in human burials that had been placed under the floors of homes.</td>
      <td>Figurines were also made of perishable materials including wood, paper, cloth, rubber, and dough.</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1990.144</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=37C443E0D334608AA0E6DEE05523F7A385065BFC2057174D434F53E84B7D6599&s=21&se=1198197559&v=&f=1990.144_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>373</th>
      <td>Seated Harlequin</td>
      <td>Heinrich Campendonk (German, 1889-1957)</td>
      <td>1922</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>Like several of his fellow Expressionists in Munich, Heinrich Campendonk believed that the natural world was a necessary antidote to a corrupt and ill society. This print portrays a horse, the recurring mystical figure of Der Blaue Reiter, along with a harlequin, a nude female in outline, a still life, and vegetation. By the 1920s, the harlequin was a ubiquitous personification of bohemian culture. Campendonk derived the hard outlines and broad, flat black planes from sources including African tribal art and Asian shadow puppetry, examples of “primitive” art forms that appealed greatly to the Expressionists’ search for authenticity.</td>
      <td>https://clevelandart.org/art/1971.245</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=92B37231D833A780BF7A41FA7A9026940E62C1A40388E9FEC9D4E7AC05443A94&s=21&se=1065507836&v=3&f=1971.245_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>374</th>
      <td>Distortion #150</td>
      <td>André Kertész (American, 1894-1985)</td>
      <td>c. 1933</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>André Kertész's Distortions series of 1933 is a unique achievement by one of the most important and inventive photographers of the 20th century. The series featured almost 200 studies of nude female figures reflected in a parabolic mirror that came from an amusement park. The inspiration for the series was a commission by the humorous and risqué French magazine Le Sourire. Using the mirror to contract, twist, and taper the nude figures, Kertész produced fantastic, ephemeral results ranging from amusing to grotesque and frightening. In this image-one of his best-Kertész created a composition that is appealing yet tantalizingly undecipherable. The model's body, simultaneously foreshortened and elongated, concrete and abstract, sweeps around mysterious reflected shapes and textures present in the studio. The Distortions have obvious affinities with the radical changes occurring in the visualization of the human figure in painting and sculpture of the 1930s. In the field of photography, however, \r\nthe series was without precedent. This rare vintage print is one of only eight (or fewer) large-scale exhibition prints produced from the Distortions series.\r\n</td>
      <td>https://clevelandart.org/art/2001.39</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=DFAD92E0B0B32D3CF4E46CA2A0D13B36EEFCB9C496D80A5FD181595CA6350500&s=21&se=1755401958&v=&f=2001.39_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>375</th>
      <td>Olympia</td>
      <td>Edouard Manet (French, 1832-1883)</td>
      <td>1867</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>When Edouard Manet’s painting Olympia was exhibited in Paris in 1865, it was met by the critics and general public with jeers, laughter, criticism, and distain. Manet had depicted his model, Victorine Meurent, as a modern day courtesan, confrontational rather than seductive. Manet’s depiction of a prostitute’s body in a contemporary setting was a radical rejection of the idealized beauty of the traditional female nude. Olympia forced recognition of troubled and contradictory attitudes toward prostitution in the mid-19th century, much to the discomfort of contemporary audiences. The artist made this etching to reproduce his controversial painting.</td>
      <td>https://clevelandart.org/art/1922.186</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9458EAFC4251FA5E03FDDCA327C427A3CB3A0E544A71A88378569FA4254A27E2&s=21&se=1049999129&v=4&f=1922.186_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>376</th>
      <td>The Penance of St. John Chrysostom</td>
      <td>Albrecht Dürer (German, 1471-1528)</td>
      <td>c. 1497</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>In the late Middle Ages an unusual legend was assigned to John Chrysostom, a hermit saint who lived in the wilderness. The story related that the daughter of the emperor lost her way during a storm and found shelter in the saint’s cave. In weakness, he betrayed his vow of chastity and in guilt, threw her off a cliff. To repent, John crawled like a beast in the wild for many years-seen at left. When the girl was miraculously found alive with her child, John was absolved of his sins.\r\n\r\nDürer’s focus on the mother and child is unprecedented in representations of the story. Although scholars generally view this print as an opportunity for the artist to depict the female nude, it is also possible that Dürer sought to illustrate a mother’s pure love and its virtuous triumph over sin.</td>
      <td>https://clevelandart.org/art/1934.339</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7282B221B395450079AA65BCE927E04764C6CFB36A25FC486E&s=21&se=1049999129&v=3&f=1934.339_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>377</th>
      <td>Seated Figurine</td>
      <td>NaN</td>
      <td>400-100 BC</td>
      <td>Central Mexico</td>
      <td>None</td>
      <td>None</td>
      <td>Clay figurines, mainly nude females, were the most common art of early villages throughout Mesoamerica. The meanings of these sweet, small,  intimate works are not known, but they are found both in human burials and in household rubbish, suggesting varied uses. Such figurines are the primary artistic context in which Mesoamerica's first great art style, the Olmec, developed.</td>
      <td>https://clevelandart.org/art/1951.104</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EB63BA0A504267D7B76AD99534BE547717374A3906D1663C64F7D016DF834174&s=21&se=1031267696&v=3&f=1951.104_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>378</th>
      <td>Renunciation (Entsagung)</td>
      <td>Max Pechstein (German, 1881-1955)</td>
      <td>1908</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>This unconventional image of a nude woman focuses not on the beauty of the female body but instead on its physical discomfort. The title of the print implies that the woman renounces the material world around her—perhaps by necessity rather than choice.  Max Pechstein and other members of the Dresden-based Expressionist group Die Brücke began to print their own lithographs in 1907. In keeping with their desire to challenge traditional print processes, they dramatically altered their drawings once on the lithographic stone. Here, Pechstein disturbed the ink by applying a wash made with turpentine, creating grainy globules across blurred and broken lines, an effect that further emphasizes the woman’s state of physical and emotional isolation.</td>
      <td>https://clevelandart.org/art/1959.328</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=CFB28CA2706BD73C9D8D828F2C4179804DE278F2A224EBFA3F3BE36CBC469E41&s=21&se=1674154316&v=2&f=1959.328_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>379</th>
      <td>Young Woman Seated on a Bed</td>
      <td>François Boucher (French, 1703-1770)</td>
      <td>before 1764</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Although this scene of a nude woman lounging in her bedchamber appears to be drawn with red chalk like the drawings by Greuze and Boucher nearby, it is actually a printed image. Chalk-manner etchings and engravings are made using special punches and rollers with teeth of various sizes set at irregular intervals to re-create the natural granular effects of chalk lines drawn on textured paper.</td>
      <td>https://clevelandart.org/art/2009.16</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A030D02C15F27611BE4661DA54C60CEBD59E047306E8432E69D27C35CAEBB744&s=21&se=1579770806&v=3&f=2009.16_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>380</th>
      <td>Yang Guifei Leaving the Bath</td>
      <td>NaN</td>
      <td>1700s</td>
      <td>China</td>
      <td>This painting relates to the legendary love story between Tang dynasty (AD 618<strong>–</strong>906) emperor Xuanzong and his consort, the beautiful Yang Guifei. Yang’s nude body can be seen through the transparent red robe as she leaves the bath. Her servant offers her a bowl of soup; cross-dressed as a male scholar, her earrings reveal that the attendant is in fact a girl.</td>
      <td>None</td>
      <td>This newly acquired painting relates to the legendary love story between Tang dynasty (618–906) emperor Xuanzong and his consort, the beautiful Yang Guifei. Set in what appears to be an 18th-century southern interior, the artist depicts Yang Guifei in a fashionable dress of semitransparent red gauze. Due to the warm and humid climate in China’s southeast, local silk workshops specialized and excelled in making airy gauze garments. Yang’s nude body can be seen through the red robe as she leaves the bath. Her servant offers her a bowl of soup. The presence of a male figure must have been a breach of customary protocol; cross-dressed as a male scholar, the earrings reveal that Yang’s attendant is in fact a girl.</td>
      <td>https://clevelandart.org/art/2017.65</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=759357B86A0AD4F480853A9B5870BA5BFD058F47EC30C914A31FFB6CA9A22832&s=21&se=1731507381&v=&f=2017.65_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>381</th>
      <td>Wave</td>
      <td>Aristide Maillol (French, 1861-1944)</td>
      <td>1895-1898</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>The color woodcuts of Katsushika Hokusai were highly esteemed. One of his most famous prints, The Great Wave off Kanagawa (1823-31), is a striking image of an enormous cresting wave. Maillol exploited the curling water motif to achieve an energetic linear design which surrounds and cushions the nude woman but also creates a lively contrast to the large, flat white shape of her body. A precursor to Maillol's future work as a sculptor, Wave reveals the artist's interest in conveying the sensual curves of the female form.</td>
      <td>https://clevelandart.org/art/1997.5</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDEAC7914554179EAD6A9F5FD1C45D1FBD069667BE4E5AFE87D&s=21&se=83547869&v=5&f=1997.5_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>382</th>
      <td>Wisdom and Destiny</td>
      <td>Henry Keller (American, 1869-1949)</td>
      <td>1913</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Keller championed modern art in Cleveland through lectures, teaching, and the example of his own work. Wisdom and Destiny, based on an essay by the Belgian writer Maurice Maeterlinck, features the two allegorical figures at the right, while a carefree shepherd at the left seems oblivious to their existence. The painting was featured in the famed Armory Show of 1913, a large-scale traveling exhibition often credited with introducing the American public to avant-garde art.\r\nBecause of the critical success surrounding Wisdom and Destiny, Keller was commissioned to create a 70-foot mural of the composition for Cleveland City Hall. However, he resentfully withdrew from\r\nthe project when asked to paint clothing on the nude figure.</td>
      <td>https://clevelandart.org/art/1928.580</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57CBEB5F2F57F10DFE4165BD75E75D70A7BB4D368322BB394D&s=21&se=1049999129&v=6&f=1928.580_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>383</th>
      <td>Curtain Panel with Scenes of Merrymaking</td>
      <td>NaN</td>
      <td>500s</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>This large panel conveys merrymaking, wealth, and power through its symbolic imagery and deep purple color. It was originally part of a luxurious wide curtain with similar panels alternating with plain linen. Although woven in Christian Egypt, pagan subjects dominate. A nude male stands beside a dancing female in transparent clothing in each of the three squares, alternating with centaurs (half-man, half-horse) in roundels. In the central square, Dionysus holds a grapevine and a weary Herakles leans on his club below. The lively animal imagery conveys wealth because few could afford such animals: lions, bulls, hares, a bear, boar, donkey, dog, and a stag.</td>
      <td>https://clevelandart.org/art/2000.5</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=419134297A23A50B9AB6624D3062185064AA17586B0948EB22BAD348D69DE90D&s=21&se=1755401958&v=5&f=2000.5_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>384</th>
      <td>Little Brother and Sister</td>
      <td>Auguste Rodin (French, 1840-1917)</td>
      <td>1916</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Rodin spent years exploring new ways of representing the human emotions of love and affection experienced at different stages of life. The bodies and gestures of the children here suggest an innocent foreshadowing of the mature couple in the artist’s celebrated sculpture <em>The Kiss </em>of 1882. Both works feature a strong tactile contrast between the rough-hewn lower area and the smooth surfaces of the nude body above. Upwardly spiraling forms, combined with shifting areas of light and shadow, infuse the sculpture with dynamic energy.</td>
      <td>https://clevelandart.org/art/1917.745</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F8BC7E8397D9E6647221D988B5063F0ACDD0325A1BCE4BA35F8117DDA0E399CD&s=21&se=754529343&v=&f=1917.745_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>385</th>
      <td>The Dream of the Doctor</td>
      <td>Albrecht Dürer (German, 1471-1528)</td>
      <td>c. 1500</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>Scholars generally agree that this curious engraving represents the contemporary proverb "Idling is the pillow of the devil," a moral message against the sin of sloth. Here, a middle-aged scholar dozes before a warm stove. The devil hovers behind him and uses a fireplace bellows to kindle impure desires. Aroused by the devil’s sinful provocation, the scholar’s subconscious conjures Venus, an object of lustful temptation.\r\n\r\nDürer’s image explores the realm of dreams and innermost thought in association with powerful female sexuality. Like many of his other engravings of this early period, Dürer uses obscure, moralistic stories to consider the female nude both creatively and intellectually. Although aimed at a primarily cultured male audience, Dürer presents Venus as a source of desire not only for the sinful idler, but for the viewer as well.</td>
      <td>https://clevelandart.org/art/1934.341</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA7242A86ECF4F1CF6163539E9EEF9FA7C92A2505B0DE66943E5&s=21&se=1049999129&v=5&f=1934.341_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>386</th>
      <td>Ornamental Shoulder Band from a Tunic</td>
      <td>NaN</td>
      <td>600-650</td>
      <td>Egypt</td>
      <td>None</td>
      <td>None</td>
      <td>Dancing human figures appear amidst leafy plants. The first and third figures are shepherds, each wearing spotted animal skin garments and holding wooden staffs. The second and fourth figures are dancers; the upper one wears a short draped garment (peplos), and the lower one appears nude except for a scarf draped over her shoulders. The brown-purple color was dyed with kermes--a bluish-red dye from dried insects--and indigo in an unsuccessful attempt to imitate purple.</td>
      <td>https://clevelandart.org/art/1919.24</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EAD82D6C274EE159F099FBC855E889F0FA40D100810A5B3C3E6FDB59E51CF8E4&s=21&se=393275125&v=2&f=1919.24_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>387</th>
      <td>Seated Figure of a Woman</td>
      <td>NaN</td>
      <td>1-200</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Although the original subject of this ancient Roman statue fragment is unknown, the cut stone at the back of its base indicates that the statue rested on a ledge, probably incorporated into a larger sculptural group. The nude upper body and relaxed pose suggest an alluring mythical woman, like Venus or a nymph, a beautiful female spirit that embodies elements of nature such as water, woods, or mountains.</td>
      <td>https://clevelandart.org/art/1920.174</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1E0711E1129993B7F2FF57210CF68B4BAF9D7AE6C60CF2733D4E8F80E1282C68&s=21&se=1310044057&v=7&f=1920.174_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>388</th>
      <td>Mr. Leroux in the Role of Alceste in Le Misanthrope</td>
      <td>Julien Vallou de Villeneuve (French, 1795-1866)</td>
      <td>mid-1850s</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Leroux was a star of the Comédie-Française, the most important theater in France. He is seen here around age 35 as the lead in Molière’s play <em>The Misanthrope</em>, a classic 17th-century play that satirizes the character flaws of the French aristocracy. As in the portrait of Bernhardt nearby, subtle aspects of pose, expression, and costuming create the character. Vallou de Villeneuve began his career as a painter and lithographer, then established himself in photography specializing in nude studies for artists, genre subjects, and portraits of actors. His prints are generally small and meant for viewing in an album rather than displaying on a wall. This portrait probably came from a theater lover’s album of French actors.</td>
      <td>https://clevelandart.org/art/1997.42</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE308A3C5307263960F98500161B2D837B06E3AC4799E3857A&s=21&se=83547869&v=3&f=1997.42_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>389</th>
      <td>Apollo the Python-Slayer</td>
      <td>Praxiteles (Greek, c. 400BC-c. 330BC)</td>
      <td>c. 350 BC</td>
      <td>Greece</td>
      <td>None</td>
      <td>None</td>
      <td>Although Praxiteles was more successful, and therefore more famous for his marble sculptures, he nevertheless also created very beautiful works in bronze. He made a youthful Apollo called the Sauroktonos (Lizard-Slayer), waiting in ambush for a creeping lizard, close at hand, with an arrow. -Pliny the Elder, 1st century ad This statue of the Apollo Sauroktonos may be the one Pliny the Elder saw in the 1st century ad. The complete sculpture most likely showed the young god pulling back a slender laurel tree with his raised left hand, while holding an arrow at waist level with his right, poised to strike the lizard creeping up the tree. Two Roman marble copies preserve the complete composition: one in the Louvre, the other in the Vatican. The museum's sculpture is the only known life-size bronze version of the Apollo Sauroktonos. Technical features such as the way the sculpture was cast and repaired in antiquity, the copper inlays of the lips and nipples, and the stone insert for the right eye (the left is a restoration) are consistent with a date in the 4th century bc. However, technically it may have been possible to produce such a work in the Hellenistic period. The Apollo Sauroktonos is thought to have been created by Praxiteles about 350 bc. Androgynous sensuality and languid, gracefully curved poses are hallmarks of his style. The finest large classical Greek statues were bronzes, but few have survived. If this sculpture is a product of Praxiteles' workshop, it is the only large Greek bronze statue that can be attributed to a Greek sculptor. Praxiteles was widely popular in his day. His famous Aphrodite of Cnidus (late 360s bc) introduced the life-size nude female figure to Western art.</td>
      <td>https://clevelandart.org/art/2004.30.a</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=525907609046730EC134CD565379F7E235BAC77B1B5A5F5E319F1599F52A6810&s=21&se=1755401958&v=8&f=2004.30.a_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>390</th>
      <td>Female Model on African Stool</td>
      <td>Philip Pearlstein</td>
      <td>1976</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>By cutting off the upper part of the model’s face, Philip Pearlstein breaks in a special way with the tradition of nude painting and portraiture. By not giving the viewer a counterpart who responds in one way or another to his or her gaze, Pearlstein focuses completely on the naked human body. By contemplating on his model’s body as a phenomenon devoid of any identity other than the attributes of sex and skin color, Pearlstein transforms it into a secluded world of forms, comparable to a natural landscape. The depiction comes across as objective and sober—there is no intimate tension or sexual charge.</td>
      <td>https://clevelandart.org/art/1978.16</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB39A6B62289D835767C3CDB8DE095451B31ABB52E2F177FB7&s=21&se=1329612074&v=2&f=1978.16_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>391</th>
      <td>Bust of a Lady (possibly Fanny Coleman)</td>
      <td>Jean-Baptiste Carpeaux (French, 1827-1875)</td>
      <td>1872</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Abandoning the strict frontality and symmetry of the neoclassical style, Carpeaux was instead inspired by French portrait busts of the 1600s and 1700s. This bust probably depicts Fanny Coleman, an English actress with whom Carpeaux may have had a romantic relationship. The turn of the head and the undulating drapery infuse the figure with a sense of drama. Patronized by Napoleon III, Carpeaux created the celebrated figural group <em>La Danse</em> for the facade of the Opera Garnier in Paris, a work renowned for its sensual nudes, but denounced by contemporaries as an affront to public decency.</td>
      <td>https://clevelandart.org/art/1975.5</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE8C10F40E03806A29636D8B542580B72640711B23BCD76FD5&s=21&se=1329612074&v=4&f=1975.5_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>392</th>
      <td>Head of a Bearded Man Gazing to His Left</td>
      <td>William Mulready (British, 1786-1863)</td>
      <td>1859</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>For Mulready, drawing was a lifelong preoccupation. These two sheets were drawn from models in the Royal Academy’s life school when the artist was 73. He was one of the Academy’s most devoted teachers, positioning the model for his students and then drawing alongside them. Closely observed and meticulously crafted, both drawings attest to Mulready’s consummate skill as a draftsman. The sensuality of the female nude and the weather-beaten face of the male model are as arresting today as they were in Victorian London.</td>
      <td>https://clevelandart.org/art/2013.241</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4DE4BF9F11F5A7CF43D51249A97D999299A0906DACEFC5690FA073924F0F3885&s=21&se=1689619227&v=&f=2013.241_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>393</th>
      <td>Ut Pictura Poesis</td>
      <td>Charles-François Hutin (French, 1715-1776)</td>
      <td>1745-1746</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Hutin’s drawing is an allegorical celebration of academic artistic training. The words <em>UT PICTURA POESIS </em>engraved on the stone tablet translate “as is painting, so is poetry.” Classical figures throughout the grand hall discuss their work as they practice different methods of making images. In the foreground, putti sculpt a portrait bust of Louis XV; behind them artists practice drawing a nude model. Among the sculptures in the room are the Farnese Hercules and the Venus de’ Medici, both famous Roman marbles in Italy, where Hutin trained from 1737 to 1742. In the upper right, Fame flies with trumpets above Minerva, the patron goddess of the arts, holding a paintbrush and palette as she drives out Ignorance and Envy.</td>
      <td>https://clevelandart.org/art/1998.76</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA722A5E992A4E799B041A24E2752A5318C17563BD4E003CB14B&s=21&se=1755401958&v=3&f=1998.76_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>394</th>
      <td>Saint Sebastian</td>
      <td>Moderno (Italian, 1467-1528)</td>
      <td>c. 1500</td>
      <td>Italy</td>
      <td>The Corinthian capital on the central column, the broken pier with reliefs of Mars and an eagle, and the pier at left with equestrian groups around an image of Venus are all ancient motifs Moderno encountered in Rome. Originally from Verona, Moderno combined this monumental classicism of Rome with his Northern Italian roots-particularly the interest in humanism he absorbed at the court of Renaissance Ferrara. Despite the torture of arrows piercing Saint Sebastian's body, the strong musculature and graceful twist of his semi-nude body reflect the Renaissance ideals of beauty befitting a holy martyr. \r\n--Hannah Segrave (October 2013)</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1992.130</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=F1223523B9D5BB8C72714AAE9BC86DE0833818ACBF58B5E274F5E818A8A934C9&s=21&se=525383189&v=4&f=1992.130_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>395</th>
      <td>Nymphs and a Satyr (Amor Vincit Omnia)</td>
      <td>Nicolas Poussin (French, 1594-1665)</td>
      <td>c. 1625-1627</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Like his colleague Claude Lorrain, Poussin depicted historical and mythological subjects in landscapes inspired by the countryside around Rome. His themes were often complex, and frequently incorporated witty allusions to classical texts. Here, a playful cupid tugs Pan, the goatlegged Greek god of the woods, toward Venus, the goddess of love. The painting cleverly illustrates the Latin phrase amor vincit omnia, or “love conquers all” (in Greek, pan means “all”). The woodland setting represents the idyllic paradise of Pan’s home.</td>
      <td>https://clevelandart.org/art/1926.26</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C9F86FF91675F6995190D4E090D9C58609842F78277EC3C2917B154C503D0D69&s=21&se=1049999129&v=7&f=1926.26_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>396</th>
      <td>Judgment of Paris</td>
      <td>Giovanni Paolo Fonduli (Italian)</td>
      <td>c. 1505</td>
      <td>Italy</td>
      <td>Fonduli's relief depicts the ancient tale of the judgment of Paris, a mythological beauty contest for three Greek goddesses judged by the Trojan prince Paris. He holds an apple in his hand while Aphrodite, Hera, and Athena stand before him. Aphrodite extends a hand, signifying her attainment of the apple, and Athena holds the flaming head of a beast in the air, suggesting the imminent conflict of the Trojan War. In some versions of the story, only Aphrodite appears naked before Paris, but in Fonduli's work, Hera and Athena are completely nude, while Aphrodite ineffectually holds light drapery to shield her lower body. Fonduli blurs the identity of the three goddesses, showing that there is little difference in their respective beauty. \r\n--Molly Bloom (August 2013)</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1984.54</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDE07314D455915364F18E3A3793F7312BA34E8465FFE860FAF&s=21&se=951169631&v=2&f=1984.54_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>397</th>
      <td>Apollo the Python-Slayer</td>
      <td>Praxiteles (Greek, c. 400BC-c. 330BC)</td>
      <td>c. 350 BC</td>
      <td>Greece</td>
      <td>None</td>
      <td>None</td>
      <td>Although Praxiteles was more successful, and therefore more famous for his marble sculptures, he nevertheless also created very beautiful works in bronze. He made a youthful Apollo called the Sauroktonos (Lizard-Slayer), waiting in ambush for a creeping lizard, close at hand, with an arrow. -Pliny the Elder, 1st century ad This statue of the Apollo Sauroktonos may be the one Pliny the Elder saw in the 1st century ad. The complete sculpture most likely showed the young god pulling back a slender laurel tree with his raised left hand, while holding an arrow at waist level with his right, poised to strike the lizard creeping up the tree. Two Roman marble copies preserve the complete composition: one in the Louvre, the other in the Vatican. The museum's sculpture is the only known life-size bronze version of the Apollo Sauroktonos. Technical features such as the way the sculpture was cast and repaired in antiquity, the copper inlays of the lips and nipples, and the stone insert for the right eye (the left is a restoration) are consistent with a date in the 4th century bc. However, technically it may have been possible to produce such a work in the Hellenistic period. The Apollo Sauroktonos is thought to have been created by Praxiteles about 350 bc. Androgynous sensuality and languid, gracefully curved poses are hallmarks of his style. The finest large classical Greek statues were bronzes, but few have survived. If this sculpture is a product of Praxiteles' workshop, it is the only large Greek bronze statue that can be attributed to a Greek sculptor. Praxiteles was widely popular in his day. His famous Aphrodite of Cnidus (late 360s bc) introduced the life-size nude female figure to Western art.</td>
      <td>https://clevelandart.org/art/2004.30.c</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=D32CDCDF4C47E50FD111E4382054582C24A211998DEB73A68E5DE002684D83EE&s=21&se=1755401958&v=6&f=2004.30.cdet01_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>398</th>
      <td>Codex Artaud XXI</td>
      <td>Nancy Spero (American, 1926-2009)</td>
      <td>1972</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Spero’s Codex Artaud is comprised of a series of 34 scrolls uniting texts by Antoine Artaud, the French actor, playwright, and poet of highly allusive writings, with Spero’s decidedly personal, feminist imagery. Codex Artaud XXI presents an extract from one of Artaud’s poems arranged in a pristine array of typed capital letters. Spero’s graphic additions include two converging cross-hatched triangles, a tiny woman riding a rat, and a heroic male nude holding a sword. The male figure, which occupies the bottom of the sheet, references Benvenuto Cellini’s sculpture Perseus Beheading Medusa (1545-54), a quintessential Renaissance subject concerned with the silencing of a powerful woman. By combining the writings of a poet who was considered an outcast and a madman with a specifically female pictorial language, Spero created a body of work that was to inspire future generations of feminist artists.</td>
      <td>https://clevelandart.org/art/2009.270</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=36B59A5C833AF0E374FD8F08B90B7A050EF338CA8858584337975F7A5E25364A&s=21&se=1285208684&v=2&f=2009.270_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>399</th>
      <td>Saint Sebastian</td>
      <td>James Barry (Irish, 1741-1806)</td>
      <td>about 1776</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This etching by Irish artist James Barry portrays Sebastian, the Roman solider who was shot with arrows for refusing to renounce his Christian faith. Following tradition, Barry depicted the martyr nude except for a loincloth and bound to a tree. Unconventionally, Barry did not depict the saint gazing heavenward as a sign of dedication to his faith, but instead masked Sebastian’s eyes in shadow, heightening the sense of physical suffering. Saint Sebastian’s idealized body, with a curved form that echoes the tree, reflects the influence of Michelangelo, whose works Barry admired during a sojourn in Rome from 1765 to 1771. This is the only known impression of this print.</td>
      <td>https://clevelandart.org/art/2016.297</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=19F7D128D879D9261C8F81E1AC2CA499D5E39A6C4AAB97D87AF58603BBF88D1C&s=21&se=1526405143&v=&f=2016.297_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>400</th>
      <td>The Swing</td>
      <td>Jean-Honoré Fragonard (French, 1732-1806)</td>
      <td>1782</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>This large print, based on a painting by Fragonard, symbolizes the pleasure-seeking and frivolous aspects of Rococo art. Unbeknownst to the man pushing the swing, a suitor reclining in the bushes gets a glimpse under the woman's skirts as she flies through the air, losing her shoe. Contemporary viewers would have understood the association of the lost shoe with sexual dalliance, a motif reinforced by other elements within the image-such as the cavorting nude figures on the base of the statue of a cupid who gestures "hush."  \r\nDelaunay's work is extremely successful in translating the qualities of a painting into the more restricted vocabulary of graphic techniques. He masterfully transposed Fragonard's charming composition, retaining all of the movement and verve of the oil. Using a rich variety of hatchings, crosshatchings, and dots, Delaunay conveyed the tonal range, lighting, and spatial effects of the painting remarkably well.\r\n</td>
      <td>https://clevelandart.org/art/1923.580</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=A1D29766D28121A59EAA0C9E83CE63071C4CCCF16CDE43E9D24F643FFD704BB1&s=21&se=1049999129&v=&f=1923.580_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>401</th>
      <td>Self-Portrait (with Head)</td>
      <td>John Coplans (British, 1920-2003)</td>
      <td>1988</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>In 1979, while serving as director of the Akron Art Museum, John Coplans worked after hours on photographs of his own feet, hands, and face. This preoccupation led him to make portraits of his own nude body. He made these images in his studio, posing himself under lights against a blank white wall, while observing the positions of his body on a video monitor. When he found what he wanted, a studio assistant made an exposure on Polaroid negative film. The process continued in the darkroom, where Coplans refined the image by cropping and then printing it in large scale. In this self-portrait, he joined his aging, sagging body with that of a youthful female, creating a humorous composite figure.</td>
      <td>https://clevelandart.org/art/1991.321</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=47099380FD68FF98A27C4B46523AA036E65C8099D076B2BD93A8D79FFEDBF242&s=21&se=1781306598&v=3&f=1991.321_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>402</th>
      <td>Venus Reclining in a Landscape</td>
      <td>Giulio Campagnola (Italian, 1482-1515)</td>
      <td>c. 1508-1509</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Campagnola introduced stipple, a technique by which shading is created with dots produced by the point of the graver, an innovation that allowed for a much greater range of tone from white to black and a more controllable graduated transition between the lighter and darker areas of the image. Stipple helped Campagnola achieve the effect of <em>sfumato</em>, subtle gradations of light and dark to model figures, which Giorgione was utilizing in painting at the time. The influence of and perhaps even the engraver’s collaboration with Giorgione is reflected in the extraordinary beauty and refinement of this rare early impression of <em>Venus Reclining in a Landscape</em>.</td>
      <td>https://clevelandart.org/art/1931.205</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0B0CC1D3CFE62C9E93724881E78E0BCF3B7D26B73509E745283A17E40ACF2DA3&s=21&se=118143847&v=&f=1931.205_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>403</th>
      <td>Bacchante and Infant Faun</td>
      <td>Frederick William MacMonnies (American, 1863-1937)</td>
      <td>1894</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>This sculpture is a smaller version of a three-quarter life-size bronze given by the artist to Charles McKim, the architect for the Boston Public Library who placed the statue in the fountain of the library's courtyard. Although the vast majority of Bostonians supported the statue, a small but powerful group denounced it as "debauched," "vice-ridden," and "treason to purity and sobriety and virtue, and Almighty God." The opponents included the Boston Brahmins—a group of Anglo-Saxon, Protestant elite—and various religious, temperance, and antivice organizations. Feeling threatened by the swell of immigrants and by the changing roles of women, these groups hoped that the library would offer an alternative to the saloons frequented by Boston's lower classes. But a statue of a nude woman reveling under the effects of wine from the Roman god Bacchus only devalued this bastion of civilization. Consequently, the Boston Art Commission rejected the statue in 1897. However, in 1992 Boston's Art and Humanities commissioner and the library trustees returned a version of the original to the courtyard without public protest.</td>
      <td>https://clevelandart.org/art/1919.589</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB324586274480C11EE0779BBBB1BEEE61DDA1AA956CF4A7EF&s=21&se=754529343&v=&f=1919.589_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>404</th>
      <td>Man Entwined by Two Snakes</td>
      <td>Giovanni Antonio da Pordenone (Italian, 1483/84-1539)</td>
      <td>c. 1527</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Although not exact copies, the compositions of both this bronze plaque and drawing derive from the Laocoön group, an ancient marble sculpture unearthed in 1506 in Rome. The nearly life-size statue of the Trojan priest Laocoön and his sons battling giant sea snakes quickly became a source of inspiration for artists. They especially appreciated the emotional anguish and physical strain portrayed by the struggling male nudes. In The Flagellation, the sculptor Moderno adopted Laocoön’s pose and muscularity for the suffering figure of Christ, thereby presenting him as an athletic and virtuous hero. Pordenone’s drawing of a man entwined by two serpents seems to be his own expressive version of Laocoön.</td>
      <td>https://clevelandart.org/art/1944.475</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C88D54049EE977BCC245F0C13EF58750853AAD8555D0309F7843AA65B81823EE&s=21&se=1031267696&v=&f=1944.475_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>405</th>
      <td>Ohio 1 (Museum of Contemporary Art Cleveland)</td>
      <td>Spencer Tunick (American, b. 1967)</td>
      <td>2004</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Since the early 1990s, Tunick has organized more than 75 temporary installations in public spaces worldwide. He deftly positions dozens, hundreds, or thousands of nude volunteers to define and expand his chosen locations, and then takes photographs. In Cleveland on June 26, 2004, Tunick created his largest North American installation to date. As part of an exhibition of his work at the Museum of Contemporary Art Cleveland (MOCA), Tunick positioned more than 2,700 people along East 9th Street. The figures rhythmically define this urban space while adding a sense of humanity. The project was a compelling union of sculpture, performance, and public art beautifully captured in this striking color image. The elongated, abstract mass of flesh challenges one’s notion of nudity and privacy.\r\nGift of Timothy and Nancy Callahan, Stewart and Donna Kohl, and\r\nMark Schwartz and Bettina Katz 2004.69</td>
      <td>https://clevelandart.org/art/2004.69</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=E48A20131668BFAAAF0A0CB14DFFD8B404B7363601B6FFCD5B8434B2994E646C&s=21&se=1755401958&v=4&f=2004.69_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>406</th>
      <td>The Three Graces</td>
      <td>Giovanni Antonio Pellegrini (Italian, 1675-1741)</td>
      <td>1786</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Since ancient times, art critics had praised painters for their ability to portray subjects so lifelike that they might be mistaken for the real thing. The mastery of printing techniques that produced full-color images like this one featuring female nudes with remarkably naturalistic skin gave French printmakers new capabilities that for centuries had been the exclusive domain of painters.</td>
      <td>https://clevelandart.org/art/1977.94</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=8147802C40E0A9C5E9A482C688D34B1E5E724CDE33D4A12438483910FFD5AE7C&s=21&se=1329612074&v=3&f=1977.94_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>407</th>
      <td>Cameo, No. 1 (Mother and Child)</td>
      <td>James McNeill Whistler (American, 1834-1903)</td>
      <td>c. 1890-1893</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Among a group of etchings Whistler made from a nude or partly draped model around 1890 was a pair entitled Cameo No. 1 and Cameo No. 2, showing a young mother playing with her child. Here, the mother leans over the child in an enclosed, protective gesture, perhaps whispering a lullaby. Whistler's treatment of the figure's robe, a semi-classical design based on Roman dress, captured the translucent qualities of the drapery which partly reveals and partly conceals the form beneath. Of this group, the artist's favorite etching was Cameo No. 1, and in 1893 he chose it to be exhibited among a selection of his prints at the Chicago World's Fair. American collectors, however, found Cameo No. 1 too risqué, and the artist was told it was unsalable because of "the thinness of the drapery."</td>
      <td>https://clevelandart.org/art/1923.29</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=09B40E5A0083DE7DB389AA5ECC4493C4587D8836E3427F6BE7443AF425134F20&s=21&se=1049999129&v=3&f=1923.29_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>408</th>
      <td>From or by Marcel Duchamp or Rrose Sélavy</td>
      <td>Marcel Duchamp (French, 1887-1968)</td>
      <td>1935-1940, 1963-1966 (Series F)</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>A leader of the Dada movement that emerged in Europe and America during World War I, Marcel Duchamp assaulted the traditional concept of art as a unique, physical object distinguished by superior handwork or craftsmanship. In its place he proposed that the central, animating principle of art should be the idea. His reputation as the century's most controversial artist began with the public's riotous reaction to his Nude Desending a Staircase at the 1913 Armory Show. That same year he created his first "ready-made" by attaching a bicycle wheel to a kitchen stool. In 1917, he exhibited a urinal under the title Fountain, and in 1919, he created his infamous L.H.O.O.Q. by adding a goatee over a printed reproduction of the Mona Lisa. Often adding inscriptions with humorous puns and ironic double meanings to his works, Duchamp raised philosophical speculation to a higher level of importance than the physical object. Shocked by his unconventional tactics, critics labeled his works"anti-art."</td>
      <td>https://clevelandart.org/art/2007.157</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0ABD03D9EB95425043E9C20259D419B41B96CBAE6698C6EE175314CF3CA6F86B&s=21&se=1828349793&v=3&f=2007.157_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>409</th>
      <td>Standing Figure of a Man</td>
      <td>Francesco di Giorgio Martini (Italian, 1439-1501)</td>
      <td>c. 1500</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Derived from classical antiquity, this nude male stands in the contrapposto pose, bearing his weight on his left leg, with his right leg bent, as if in motion. He steps lightly, so that only the toes of his right leg touch the ground. He stands on a laurel wreath, a plant which in antiquity would be waved as a symbol of joy after a military victory. During the Renaissance, small bronzes were kept in cabinets, bringing the gods and goddesses of ancient times into the home. Flaws in the bronze casting suggest that this piece is from the early Renaissance. In 1971, Ulrich Middeldorf rejected Olga Raggio's original attribution to an anonymous Florentine, attributing it instead to the sixteenth-century Sienese artist Francesco di Giorgio Martini (1439-1501), based partly on its resemblance to Francesco's bronze angels in Siena's Duomo. However, such details as the awkward posture and frozen motion of the body relate this work more closely to Francesco's pupil Giacomo Cozzarelli (1453-1515). Standing Figure of a Man is attributed to the circle of Francesco di Giorgio Martini, as uncertainties remain regarding its exact author.</td>
      <td>https://clevelandart.org/art/1947.509</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2D6571D3C7771C57B81B730CE39A58469E68CCA8234EB24216CDF0286BD4DCA3&s=21&se=1031267696&v=4&f=1947.509_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>410</th>
      <td>Virgin and Child</td>
      <td>Benedetto Buglioni (Italian, 1461-1521)</td>
      <td>1500-1510</td>
      <td>Italy</td>
      <td>This small devotional object differs stylistically from Buglioni's Madonna and Child Enthroned with Saints Francis and Anthony Abbot (1921.1180), also in the museum's collection.  In contrast to the other work, the faces of the Madonna and Child are significantly less expressive, and the passages of exposed skin have been left unglazed.  The Madonna holds a nude Christ on her left arm and wears a mantle of blue with green lining and a maroon tunic with a yellow collar and cuffs.  Her bright yellow shoes and the purple of her tunic are characteristic of Buglioni's color scheme and can also be seen in the other work.  The tabernacle frame is probably contemporary to the figures, although it may not have been created specifically for this altarpiece.  Buglioni likely worked in the studio of Andrea della Robbia, where he would have studied the family's techniques for glazing terracotta.  The frame is classically inspired, with Corinthian capitals and Buglioni's characteristic egg and dart moldings.  The facial type in this Madonna and Child recalls Buglioni's Madonna delle Grazie in Florence's Palazzo Frescobaldi, and because of this similarity, the scholar Allan Marquand attributed the Cleveland Madonna and Child to Buglioni in 1921, although this attribution has since been questioned.\r\n\r\nAna Dieglio 2009</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1916.1005</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B0ED9F5C2D97671E56B3FFCC41026A146CFB4F91B1B01258154A614876505DCA&s=21&se=156361197&v=&f=1916.1005_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>411</th>
      <td>The Dance</td>
      <td>Carl Wilhelm I Kolbe (German, 1757-1835)</td>
      <td>1799</td>
      <td>Germany</td>
      <td>None</td>
      <td>None</td>
      <td>The Dance represents a stylistic fusion of classicism and scientific naturalism that dominated landscape art at the end of the 18th century. The circle of nudes dancing to a satyr’s music, ancient ruins, and sacrificial alter evoke a classical idyll, while the surrounding landscape, particularly the tree and foreground burdocks, are rendered with botanical precision.</td>
      <td>https://clevelandart.org/art/1916.1066</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=572CB2BD5244C0B3F22725AC88908BF485975146D3BF09D38D43696040B257A8&s=21&se=156361197&v=&f=1916.1066_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>412</th>
      <td>Pair of Putti</td>
      <td>NaN</td>
      <td>late 1500s</td>
      <td>Italy</td>
      <td>These gilt bronze putti, or figures of plump male children in the nude, are near mirror images of one another.  Both are seated, with their arms flung to the side.  Minimal detailing exists in the hands and feet, with the fingers and toes signified only by simple line indentations.  The function and original placement of the putti is uncertain, but their posture and the placement of their hands suggest that originally they may have been placed in a larger statue group, perhaps holding either side of a banner, or even a piece of furniture.  A will drawn up by Guglielmo della Porta in 1558 mentions the existence of bronze Passion scenes and crucifixes, suggesting that the artist may have created other small bronze groups as well.  John Pope-Hennessey attributed these works to della Porta based on their similarity to the figures of Apollo and Artemis in Latona and her Children (Victoria and Albert Museum, London) and the monumental putti in his Tomb of Cardinal Paolo Cesi in Santa Maria Maggiore in Rome, although more recent research points away from della Porta as the maker.\r\n\r\nAna Dieglio 2009</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1968.64</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=280E3F80AC10906358CCFCD6180E7D4DD825BC7A9BFD5EBD52E8CBB30FB7ECF0&s=21&se=23404858&v=&f=1968.64_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>413</th>
      <td>Laurels Number Four: Riders in the Park</td>
      <td>Laurels Gallery</td>
      <td>1934</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Milton Avery was a quiet, humble man whose art reflected his life. Inspired by personal experience and based on direct observation, his relaxed, intimate works depict a serene, ideal world. Landscapes evoke summer trips to the country and seaside; urban scenes depict family and friends. From apparently ordinary subject matter, Avery constructed innovative works in which simplified flat forms build perfectly balanced, satisfying compositions. \r\n\r\nBetween 1947 and 1948, Chris Ritter, director of the Laurel Gallery, New York, published four issues of Laurels, portfolios of prints, literature, and original calligraphy often on paper handmade by Douglass Howell. The museum's collection already includes the first two issues, with prints by Joan Miró, Stanley William Hayter, Reginald Marsh, and others. The five drypoints shown here were made for the fourth Laurels Portfolio. They encompass Avery's favorite motifs: landscape, seascape, and figure, all delineated with amazing economy. For example, the Nude with Long Torso is drawn with two horizontal lines that span the length of the plate. By subtly controlling the thickness of line or placement of a shape, a sense of roundness, weight, and a shallow space are created.\r\n</td>
      <td>https://clevelandart.org/art/2001.9.1</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C753B6125DA56B168930A4EB40D0BD1B90D81D057754E5EB8BA5D8F098FB320F&s=21&se=1456276286&v=5&f=2001.9.1_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>414</th>
      <td>Bacchanale</td>
      <td>Malvina Hoffman (American, 1887-1966)</td>
      <td>1917</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Bacchanale\r\nDuring a visit to London in 1910, Malvina Hoffman attended two performances by the great Russian ballerina Anna Pavlova (1882-1931). The program included the "Autumn Bacchanale" section of The Seasons by Aleksandr Glazunov (1865-1936), which Pavlova danced with her partner, Mikhail Mordkin. A spirited and erotic work, the "Bacchanale" was inspired by the legendary followers of Bacchus (Dionysus in Greek), the ancient god of wine. The piece featured the two dancers dashing about the stage, their movements accented by veils of floating gauze. Hoffman later recalled being struck by "impressions of motion of a new kind, of dazzling vivacity and spontaneity and yet with a control that could come only from long discipline and dedication." \r\nIn 1912 Hoffman modeled a 14-inch composition of Pavlova and Mordkin called Bacchanale Russe (Russian Bacchanale). The smiling dancers were shown nude, holding a billowing scarf above their heads, as they made their entrance on stage. It took six weeks of modeling sessions to complete the composition. Hoffman wrote to her sister: "I have never had such a difficult problem-it's awful---I pray I may never be seized with the desire to fit two flying creatures together...again. If you could see the antics that go on, trying to get my two models to take the pose together!"\r\nNine bronze casts were made of the original version during Hoffman's lifetime. In 1917 two casts were produced in a larger, nearly six-foot size. One was placed in the Luxembourg Gardens in Paris, where it was destroyed during the German Occupation of Paris in World War II. The other found a home in the garden of the Henry G. Dalton house on Lake Shore Boulevard in Bratenahl, Ohio. It was given to the Cleveland Museum of Art in 1943. Vibrant with life, the two figures demonstrate Hoffman's ability to capture a fleeting moment of the dance in tangible, permanent form, without sacrificing the rhythmic flow of live movement.\r\nThe Sculptor and the Dancer\r\nHoffman did not actually meet Anna Pavlova until 1914, four years after she first saw the dancer perform. The two women developed a warm friendship, with Pavlova encouraging Hoffman to attend rehearsals and make sketches from the wings during performances. \r\nThey also collaborated, until Pavlova's death in 1931, on a long-term endeavor---a frieze of relief panels depicting the highpoints of the entire "Bacchanale" in sequence. Pavlova posed for the panels with partners Laurent Novikov and Alexander Volinine, but for Hoffman that was not enough. She prepared herself for the project by learning the entire dance herself and performing it with Pavlova's assistant partner, Andreas Pavley.\r\nFrom New York to Paris\r\nThe daughter of English concert pianist Richard Hoffman, Malvina Hoffman grew up in New York City. Her artistic training began with drawing and painting, but after studying at the Art Students League and receiving helpful criticism from sculptors Herbert Adams (1858-1945) and Gutzon Borglum (1867-1941), she decided to concentrate on sculpture. Traveling to Europe in 1910, she studied with Auguste Rodin (1840-1917) in Paris. To provide herself with a solid foundation in the art and craft of sculpture she took courses in dissection at the Cornell College of Physicians and Surgeons in New York and also learned the technology of casting, chasing, and patinating bronze. \r\nHoffman's interest in the ballet did not preclude production of works dealing with other subjects. She completed several public monuments and numerous portraits, which earned her an international reputation and led to her most ambitious project: the 1929 commission from Chicago's Field Museum of Natural History for more than 100 life-size figures, heads, and busts representing the peoples of the earth.</td>
      <td>https://clevelandart.org/art/1943.384</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=5B79BA773164218679F077C54599B195C704190A952195CE519FBDD2BB9A9AB1&s=21&se=1031267696&v=&f=1943.384_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>415</th>
      <td>Study after Nature</td>
      <td>Julien Vallou de Villeneuve (French, 1795-1866)</td>
      <td>c. 1853-1855</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>In the early 1850s, despite protests from moralists, photographs of nudes (known as académies) were discreetly produced under the guise of "artist's studies." Among those most skilled at making such studies was Julien Vallou de Villeneuve. A successful painter and lithographer during the 1820s and 1830s before turning to photography in the early 1840s, he created a new repertoire of poses for artists to use as compositional aids. Among his clients was the French realist painter Gustave Courbet (1819–1877).</td>
      <td>https://clevelandart.org/art/1993.163</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2B13BA90C444762B00A73D0FF823A033F35708704C5AE29D05062C9AC56A6C2B&s=21&se=1781306598&v=3&f=1993.163_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>416</th>
      <td>Hercules and the Nemean Lion</td>
      <td>Moderno (Italian, 1467-1528)</td>
      <td>16th century</td>
      <td>Italy</td>
      <td>This plaquette by Moderno, a Northern Italian sculptor active in the early 1500s, displays the ancient subject of Hercules wrestling the Nemean lion. The subject of this plaquette finds its origins in the ancient myth of the Twelve Labors of Hercules, a popular theme during the Renaissance. After killing his wife and child in a fit of rage, Hercules was sent by an oracle to be punished by King Eurystheus of Tiryns. The King presented Hercules with twelve challenges, the first of which was to kill the Nemean lion. Twice as large as a normal lion, it had an impenetrable hide, symbolic of the uncontrollability of nature. Hercules tracked the lion to its cave, and after failing to kill it with the arrows of Apollo, Hercules strangled the lion with his bare hands. As Hercules' strength allowed him to defeat the lion, the myth expresses human dominance over nature. Hercules then used the lion's razor-sharp claws to remove its skin and returned to King Eurystheus wearing the impenetrable skin as protection. To honor the lion's struggle against Hercules, Zeus transformed the lion into the constellation Leo. In this scene, the nude Hercules stands in profile facing left with his left foot resting on the plaquette's lower border. The plaquette has no background other than a barren tree against which rest Hercules's club and Apollo's quiver and bow. The lion rests its rear paw on Hercules' thigh as Hercules lifts the lion off of the ground. The lion's body and mane consists of stylized tufts of fur which contrast with the smooth, naturalistic form of Hercules' body. The minute, short curling lines of the hair and mane create a rhythmic effect while the smooth curves of the forms place an emphasis on contour-this contrast of visual rhythm with smooth form is in fact characteristic of Moderno's bronze relief sculpture. The composition meanwhile has its origin in ancient Greek coins as well as Renaissance cameos.</td>
      <td>None</td>
      <td>Whereas the ancient sculpture of Hercules nearby may have served as a sacred offering, this Renaissance plaquette and statue were collected and appreciated for the sake of art. After strangling the Nemean lion, Hercules kept its pelt; the hero often wears it in depictions of other legends. Ancient Greek coins likely provided the model for this particular composition, in which Hercules lifts the lion off the ground as it claws at his torso and leg.</td>
      <td>https://clevelandart.org/art/1967.135</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDED433A57AC01E2C8FFA2C56B7FEA71333544F6B88618C0C9F&s=21&se=172172604&v=4&f=1967.135_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>417</th>
      <td>Putto</td>
      <td>NaN</td>
      <td>late 1500s</td>
      <td>Italy</td>
      <td>These gilt bronze putti, or figures of plump male children in the nude, are near mirror images of one another.  Both are seated, with their arms flung to the side.  Minimal detailing exists in the hands and feet, with the fingers and toes signified only by simple line indentations.  The function and original placement of the putti is uncertain, but their posture and the placement of their hands suggest that originally they may have been placed in a larger statue group, perhaps holding either side of a banner, or even a piece of furniture.  A will drawn up by Guglielmo della Porta in 1558 mentions the existence of bronze Passion scenes and crucifixes, suggesting that the artist may have created other small bronze groups as well.  John Pope-Hennessey attributed these works to della Porta based on their similarity to the figures of Apollo and Artemis in Latona and her Children (Victoria and Albert Museum, London) and the monumental putti in his Tomb of Cardinal Paolo Cesi in Santa Maria Maggiore in Rome, although more recent research points away from della Porta as the maker.\r\n\r\nAna Dieglio 2009</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1968.64.1</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FFF48807526AF867DDDBCF601184034D5379FCAF32335B718D&s=21&se=1906185785&v=5&f=1968.64.1_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>418</th>
      <td>Laurels Number Four: March at a Table</td>
      <td>Laurels Gallery</td>
      <td>1948</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Milton Avery was a quiet, humble man whose art reflected his life. Inspired by personal experience and based on direct observation, his relaxed, intimate works depict a serene, ideal world. Landscapes evoke summer trips to the country and seaside; urban scenes depict family and friends. From apparently ordinary subject matter, Avery constructed innovative works in which simplified flat forms build perfectly balanced, satisfying compositions. \r\n\r\nBetween 1947 and 1948, Chris Ritter, director of the Laurel Gallery, New York, published four issues of Laurels, portfolios of prints, literature, and original calligraphy often on paper handmade by Douglass Howell. The museum's collection already includes the first two issues, with prints by Joan Miró, Stanley William Hayter, Reginald Marsh, and others. The five drypoints shown here were made for the fourth Laurels Portfolio. They encompass Avery's favorite motifs: landscape, seascape, and figure, all delineated with amazing economy. For example, the Nude with Long Torso is drawn with two horizontal lines that span the length of the plate. By subtly controlling the thickness of line or placement of a shape, a sense of roundness, weight, and a shallow space are created.\r\n</td>
      <td>https://clevelandart.org/art/2001.9.4</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C753B6125DA56B16668D382E25B509949BB5BE97AF756C1142672CA51C558719&s=21&se=1456276286&v=4&f=2001.9.4_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>419</th>
      <td>Ohio 4 (Museum of Contemporary Art Cleveland)</td>
      <td>Spencer Tunick (American, b. 1967)</td>
      <td>2004</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Since 1994, Tunick has organized over sixty-five temporary site-related installations in public places worldwide. He deftly positions nude male and female volunteers to define and expand his chosen locations, then photographs and videotapes the installations. In early 2004, the Museum of Contemporary Art Cleveland organized an exhibition surveying Tunick's photographs in preparation for his June 26, 2004, installation in Cleveland. The shoot involved some 2,754 individuals-the largest installation in North America. For Ohio 4 (Museum of Contemporary Art Cleveland), he posed just the men-all 1,387-kneeling in Voinovich Park, North Coast Harbor, at the end of East 9th Street. This outstanding image showcases Tunick's impressive creative and aesthetic talents to orchestrate and record a complex public installation. The undulating figures rhythmically define the vast urban\r\nspace while adding a haunting sense of humanity. The color photograph beautifully captures a compelling union of sculpture, performance art, and public art, with Cleveland Browns Stadium punctuating the bright, early morning sky in the background.\r\n</td>
      <td>https://clevelandart.org/art/2005.346</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=ACFBB70482EEA28ED7B05EF3EE3B7BD0FB5095D4A1DEAA30BDDD727DD7926683&s=21&se=1755401958&v=4&f=2005.346_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>420</th>
      <td>Laurels Number Four: Umbrella by the Sea</td>
      <td>Laurels Gallery</td>
      <td>1948</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Milton Avery was a quiet, humble man whose art reflected his life. Inspired by personal experience and based on direct observation, his relaxed, intimate works depict a serene, ideal world. Landscapes evoke summer trips to the country and seaside; urban scenes depict family and friends. From apparently ordinary subject matter, Avery constructed innovative works in which simplified flat forms build perfectly balanced, satisfying compositions. \r\n\r\nBetween 1947 and 1948, Chris Ritter, director of the Laurel Gallery, New York, published four issues of Laurels, portfolios of prints, literature, and original calligraphy often on paper handmade by Douglass Howell. The museum's collection already includes the first two issues, with prints by Joan Miró, Stanley William Hayter, Reginald Marsh, and others. The five drypoints shown here were made for the fourth Laurels Portfolio. They encompass Avery's favorite motifs: landscape, seascape, and figure, all delineated with amazing economy. For example, the Nude with Long Torso is drawn with two horizontal lines that span the length of the plate. By subtly controlling the thickness of line or placement of a shape, a sense of roundness, weight, and a shallow space are created.\r\n</td>
      <td>https://clevelandart.org/art/2001.9.3</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C753B6125DA56B16285EC86430585C7D4B1D87479036E4AEF71A10FC3A842389&s=21&se=1755401958&v=5&f=2001.9.3_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>421</th>
      <td>Laurels Number Four: Head of a Man</td>
      <td>Laurels Gallery</td>
      <td>1935</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Milton Avery was a quiet, humble man whose art reflected his life. Inspired by personal experience and based on direct observation, his relaxed, intimate works depict a serene, ideal world. Landscapes evoke summer trips to the country and seaside; urban scenes depict family and friends. From apparently ordinary subject matter, Avery constructed innovative works in which simplified flat forms build perfectly balanced, satisfying compositions. \r\n\r\nBetween 1947 and 1948, Chris Ritter, director of the Laurel Gallery, New York, published four issues of Laurels, portfolios of prints, literature, and original calligraphy often on paper handmade by Douglass Howell. The museum's collection already includes the first two issues, with prints by Joan Miró, Stanley William Hayter, Reginald Marsh, and others. The five drypoints shown here were made for the fourth Laurels Portfolio. They encompass Avery's favorite motifs: landscape, seascape, and figure, all delineated with amazing economy. For example, the Nude with Long Torso is drawn with two horizontal lines that span the length of the plate. By subtly controlling the thickness of line or placement of a shape, a sense of roundness, weight, and a shallow space are created.\r\n</td>
      <td>https://clevelandart.org/art/2001.9.2</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=C753B6125DA56B16B0AFD5C58D7E85B28B10AFF73C11D36D7E8DAC755F99B87C&s=21&se=1755401958&v=4&f=2001.9.2_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>422</th>
      <td>Figure</td>
      <td>Gaston Lachaise (American, 1882-1935)</td>
      <td>c. 1924-26</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Lachaise's highly personal sculptures of voluptuous nudes, inspired by his wife Isabel Nagel, display his idealized conception of the female figure with full, rounded breasts, arms, and thighs surrounding a delicate, slender waist. The artist's drawings, in which a single unmodulated line describes contour, are spontaneous and direct. Although this drawing was not a study for a sculpture, the figure has a three-dimensional quality suggested by the changes in the width of the line. Lachaise's drawings, like his sculptures, are dazzling combinations of force and grace, simultaneously heroic and erotic.</td>
      <td>https://clevelandart.org/art/1964.465</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=9E450695E4D7932ED2252326125B00950A1C827258D10130F814C89DC68A76D0&s=21&se=1674154316&v=&f=1964.465_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>423</th>
      <td>Stems</td>
      <td>Lee Friedlander (American, b. 1934)</td>
      <td>1994</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>For more than forty years, Lee Friedlander has created "social landscapes," images of jazz musicians, monuments, city streets, suburban backyards, and factory workers. Yet his portraits of family and friends, nudes, and still lifes demonstrate his versatility. This photograph, depicting the stems of cut flowers in a reflective glass bowl, comes from a series that Friedlander made while recovering from knee surgery. Through his keen powers of observation and conceptualization, he transformed the stems into a striking abstract composition of bold geometric forms, bathed in warm, atmospheric light. Aspects of the photographer's distinctive style-a close-up viewpoint, foreshortening, compressed space, and layering-appear in this work.</td>
      <td>https://clevelandart.org/art/1999.257</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=45769A39F7A6267F44D9336B5E9FF6ED41C7DF121B66B68EDA50E8EEC5D8B6DF&s=21&se=1755401958&v=5&f=1999.257_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>424</th>
      <td>The Wrath of Neptune</td>
      <td>Tiziano Minio (Italian, 1511/12-1552)</td>
      <td>late 1500s to early 1600s</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>In Roman mythology, Neptune (or Poseidon in Greek mythology) is the god of flowing water and is believed to protect against droughts. This small bronze is likely a sculptural representation of the Wrath of Neptune from the Aeneid. During the Trojan War, Juno incited winds to sink Trojan ships. Infuriated by Juno's incursion into his domain of the sea, Neptune rides out in his chariot, saving the Trojan army and threatening to punish the winds for their insolence. The massive, muscular nude male figure stands upright on his seahorse-drawn chariot, twisting to face the wind. The turbulent waves and the windblown hair and beard of the figure give life to this small bronze. The figure's left hand would have originally held reins, while the right hand, which once held a trident, is outstretched in the act of commanding the waves to be still. This bronze was once attributed by art historian Wilhelm von Bode to Jacopo Sansovino based on its similarity to known works, such as the over-life-size marble figure of Neptune of the Scala di Giganti outside of the Doge's Palace in Venice. However, by 1921 Leo Planiscig had noted the CMA Neptune's similarity to bronzes and reliefs by Sansovino's pupil, Tiziano Minio, in the Loggetta at San Marco Square, Venice. In Minio's Jupiter on the Island of Candia for the Loggetta, the waves are represented with the same incised, undulating grooves as in this work. Neptune possesses the same stocky proportions and heavy musculature which was characteristic of Minio. The high polish and sweeping contours of the figure are characteristic of Sansovino, but the heightened drama of the subject display Minio's stylistic advancement from his teacher.</td>
      <td>https://clevelandart.org/art/1974.273</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FFAEE1F5759F6DDE6F0B55C149EF10646DCF54E37566F0E5B3&s=21&se=1065507836&v=4&f=1974.273_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>425</th>
      <td>Love Requests Venus to Return His Weapons to Him</td>
      <td>François Boucher (French, 1703-1770)</td>
      <td>1768</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>For his aristocratic patrons, Boucher made numerous paintings of intriguing female nudes, thinly veiled as Roman goddesses lounging in luxury. Bonnet’s color chalk-manner prints of similar subjects offered middle-class buyers sumptuous yet affordable versions of Boucher’s wildly popular images. Here, a tiny cupid pleads with Venus for his quiver of arrows as she gazes teasingly at the viewer. Their feathered ends simultaneously conceal and draw attention to her nudity. With cupid’s arrows in her control, it is Venus—rather than cupid—who has the power to arouse feelings of love.</td>
      <td>https://clevelandart.org/art/2006.182</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2FFF96D56F26D7BB21932F5C777096947B03105AA57739649648C57F0FFA5C51&s=21&se=1828349793&v=&f=2006.182_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>426</th>
      <td>The Oath of the Seven Chiefs against Thebes</td>
      <td>Anne-Louis Girodet de Roucy-Trioson (French, 1767-1824)</td>
      <td>c. 1800</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Girodet found inspiration for this drawing in Aeschylus’s Greek tragedy <em>Seven against Thebes.</em> Dramatized with powerful physicality, seven warrior leaders from Argos raise weapons to the war deities Ares and Enyo at the far left as they immerse their hands in the blood of a sacrificed bull, and swear an oath to defeat Thebes. Girodet’s strong black outlines and idealized male nudes are characteristic of Neoclassicism’s calculated restraint. Yet the flash of lightning and the warrior’s impassioned expressions intensify the emotional and psychological content of the scene, anticipating the growth of romanticism in European art during the early 1800s.</td>
      <td>https://clevelandart.org/art/2000.71</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=2FFF96D56F26D7BB691BFA2520D6348624CB2AB4D8FBFF6C6CCF8E4513204D5A&s=21&se=1456276286&v=&f=2000.71_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>427</th>
      <td>The Birth of Venus</td>
      <td>Raphael (Italian, 1483-1520)</td>
      <td>c. 1516</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Marco Dente was one of several printmakers who re-created Raphael’s designs as engravings. Both of Dente’s compositions after Raphael (see also the engraving above) relate to the latter’s frescos in the <em>stufetta</em>, or bathroom, of Cardinal Bernardo Dovizi da Bibbiena’s apartments in the Vatican Palace. A scholar of classical antiquity and a patron of the arts, the Cardinal personally chose these mildly erotic female nudes, a common subject in bathroom decorations, even for members of the clergy. Depicted here is Venus, the goddess of love and fertility. According to the myth of her birth, she was created in sea foam from the castrated genitals of the primeval sky god Uranus. In the clouds, Uranus’s son Saturn, the god of time, prepares to mutilate his father with a curved sword. Meanwhile, Venus steps out of frothy waves onto a seashell.</td>
      <td>https://clevelandart.org/art/1923.1101</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=AF226C61E597B26CBEDF9A212EFBD3F6C7DEABFA5B6AE02A78C2BD249C4F8681&s=21&se=1049999129&v=3&f=1923.1101_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>428</th>
      <td>Buddha</td>
      <td>Kiyoshi Saito (Japanese, 1907-1997)</td>
      <td>1951</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>In 1951 Saito began a series of prints based on ancient burial figures called haniwa, terra-cotta funerary statuettes with a charming, naïve naturalism from the 3rd to 6th century. Saito found them to "have a beauty like nudes," and using woodcut, he reproduced in prints the rough surface of the original clay Buddha.\r\n\r\nIn 1985, following the death of her husband, Eleanor Smith gave the museum his collection of 198 color woodcuts by Saito, nearly the complete oeuvre of this well-known, popular printmaker.</td>
      <td>https://clevelandart.org/art/1985.377</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=1F448EBE0D6107CFABB17D41DE662A53A0963D5582972F9F514B8B60BA6A80EB&s=21&se=92257091&v=&f=1985.377_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>429</th>
      <td>The Flagellation</td>
      <td>Moderno (Italian, 1467-1528)</td>
      <td>16th century</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>Although not exact copies, the compositions of both this bronze plaque and drawing derive from the Laocoön group, an ancient marble sculpture unearthed in 1506 in Rome. The nearly life-size statue of the Trojan priest Laocoön and his sons battling giant sea snakes quickly became a source of inspiration for artists. They especially appreciated the emotional anguish and physical strain portrayed by the struggling male nudes. In The Flagellation, the sculptor Moderno adopted Laocoön’s pose and muscularity for the suffering figure of Christ, thereby presenting him as an athletic and virtuous hero. Pordenone’s drawing of a man entwined by two serpents seems to be his own expressive version of Laocoön.</td>
      <td>https://clevelandart.org/art/1967.150</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=0970074B662487FF2E3C89617E33132C0489C952BDC4CA8F7021A238F09EEEA3&s=21&se=172172604&v=4&f=1967.150_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>430</th>
      <td>The Ecstasy of Mary Magdalene</td>
      <td>Albrecht Dürer (German, 1471-1528)</td>
      <td>1501-1504</td>
      <td>England</td>
      <td>None</td>
      <td>None</td>
      <td>Dürer’s woodcut of Mary Magdalene represents a popular subject in German art and is considered a schlechtes Holzwerk, a simple woodcut intended for a general audience. According to a medieval book of saints’ lives known as the Golden Legend, Mary Magdalene spent the last 30 years of her life as a hermit outside of Marseilles, France, where she was miraculously borne aloft to heaven seven times a day to hear the choir of angels. Considered a fallen woman in her early life, Mary earned redemption through her complete devotion to Christ.\r\n\r\nDuring this period, Dürer was preoccupied with the laws of human proportion and the female figure. Mary Magdalene’s powerful legs and widened hips are comparable to the female nudes in Dürer’s The Dream of the Doctor and Adam and Eve.</td>
      <td>https://clevelandart.org/art/1941.42</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=EC430C4C22E2AA724E4849FA276588AEA13B5C1E50E7E230E1378C69C460F8D2&s=21&se=1031267696&v=3&f=1941.42_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>431</th>
      <td>Portrait of Anne Vallayer-Coster</td>
      <td>François Dumont (French, 1751-1831)</td>
      <td>1804</td>
      <td>France</td>
      <td>Born in 1744, Anne Vallayer-Coster was a prominent still-life painter in France during the 1770s and 1780s. The daughter of a goldsmith, she grew up observing the intricacies of metalwork, acquiring an attention to detail apparent in her paintings. She was apprenticed to the seascape painter Joseph Vernet who encouraged her to pursue painting as a profession. In 1770 Vallayer-Coster sought membership in the exclusive French Académie Royale de Peinture et de Sculpture with the submission of two still-life paintings, "The Attributes of Painting, Sculpture, and Architecture" and "The Attributes of Music." She was accepted with full rights and privileges to the overwhelmingly male academy-the only woman to be granted membership during this period independent of royal support or ties to a male academician. Denis Diderot, a French art critic and philosopher, responded to her submissions: "It is certain that if all new members made a showing like Mademoiselle Vallayer's, and sustained the same high level of quality there, the Salon would look very different!" Since women were officially barred during the 18th century from studying the male nude from life, still-life painting was among the few genres open to female painters. Garnering a reputation as a beautiful socialite who possessed the painting skills usually ascribed to male history painters, she was the only woman to enjoy the honor of lodging in the Louvre under the invitation of Marie Antoinette, who greatly admired her work. Marie Antoinette herself was present at the artist's 1781 marriage to Jean-Pierre-Silvestre Coster in the halls of Versailles. Despite the close associations with Antoinette and her administration, Vallayer-Coster survived the French Revolution of 1789 and the Reign of Terror in 1793. Many of her patrons were not so fortunate, and the artist's production virtually stopped until 1804, when François Dumont painted this miniature. During this period, Vallayer-Coster struggled to reestablish herself among the elite of Paris. Over the course of 1804, she exhibited two works at the Salon and had two works commissioned by Empress Josephine de Beauharnais. Dumont's miniature depicts Vallayer-Coster holding a paintbrush in her right hand and a vase with flowers in her left, clearly an allusion to her skill as a still-life painter. Perhaps Vallayer-Coster requested that Dumont portray her as an artist in order to reassert her status as an important still-life painter. She may have intended to give the miniature to the empress in gratitude for her commissions and in the hope that there would be more to come. Her last major work, "Still Life with Lobster", was given to Louis XVIII and displayed at the Salon in 1817. Vallayer-Coster died in 1818 at the age of 74. Dumont worked closely with Marie Antoinette and, like Vallayer-Coster, grappled with diminished patronage after the queen's death and years of revolution. He attempted to reestablish his practice during the reigns of the Bourbon kings Louis XVIII and Charles X, but the competition provided by miniaturists like Jean-Baptiste Isabey (1767-1855) proved too strong. Dumont's exquisite portrait of Vallayer-Coster was produced during a time when both artist and sitter had fallen from the limelight and were struggling to prove their enduring relevance in the new regime. Ashley Bartman (May 2014)</td>
      <td>None</td>
      <td>None</td>
      <td>https://clevelandart.org/art/1943.639</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=B06DDFF2916C41CB6CEBF89253419ED64592F72CEE8729BD55CFFF84CDD239BF&s=21&se=1031267696&v=5&f=1943.639_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>432</th>
      <td>Construction of an Elevated Railway:  Bridge over the Cours de Vincennes</td>
      <td>Paul Désiré Trouillebert (French, 1829-1900)</td>
      <td>1888</td>
      <td>France</td>
      <td>None</td>
      <td>None</td>
      <td>Conceived in 1851, after Napoleon III came to power, the railway encircling Paris was intended to transport merchandise and, eventually, passengers. The railway represented a new convenience, but measures were needed to ensure the safety of other traffic. The numerous railway crossings included in the initial plans turned out to be a source of fatal accidents. To remedy the problem, the platforms and retaining walls were to be raised at the most dangerous spots. The Cours de Vincennes, in the eastern part of Paris, had been one of the deadliest intersections. The work on an elevated railway bridge over this street, which is depicted here, was completed in February 1889. Trouillebert concentrated on portraits until about 1881, when he began to focus on landscapes. He also painted everyday scenes and nudes. He was commissioned by Edme Piot, a public works contractor, to paint this and four related views of the Paris railway construction.</td>
      <td>https://clevelandart.org/art/1980.289</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=4B32F978B28D5E52E80EAB60A78551C5DA7113C5F24D0566AF60DE7FF786183E&s=21&se=1127467748&v=4&f=1980.289_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>433</th>
      <td>Photographer (Bill Brandt)</td>
      <td>Nancy Hellebrand (American, 1944-)</td>
      <td>1973</td>
      <td>America</td>
      <td>None</td>
      <td>None</td>
      <td>Philadelphia native Nancy Hellebrand, recognized for her portraits of the British working class, was strongly influenced by photographer Bill Brandt (1904-1983), with whom she studied from 1971 to 1973. Brandt's work included social documentation, portraiture, landscape, and nudes. While in Paris during the late 1920s, his association with artists of the Surrealist movement helped shape his sensibilities. Alluding to that movement, Hellebrand made reference to the eye-a recurrent symbol in Surrealist art-by cropping one side of the sitter's face. She emphasized Brandt's surroundings as a means of identifying his artistic and intellectual pursuits. Although positioned at the edge of the image, the sitter's placement at the forefront and Hellebrand's use of intense light and shadow produced a wonderful field of depth. Brandt's somber, isolated, and perhaps introspective appearance is reinforced by the portrait's stark black-and-white tones.</td>
      <td>https://clevelandart.org/art/1991.299</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=47099380FD68FF986602C642B64B05DA4716E2C3684F285936C9729C4DF57823&s=21&se=264312387&v=3&f=1991.299_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>434</th>
      <td>Woman Bathing Her Feet in a Brook</td>
      <td>Rembrandt van Rijn (Dutch, 1606-1669)</td>
      <td>1658</td>
      <td>Netherlands</td>
      <td>None</td>
      <td>None</td>
      <td>Since the great majority of Rembrandt's works are portraits or depictions of biblical and mythological subjects, human figures are an important element of his oeuvre. During the course of a long career Rembrandt produced nine etchings of nudes, conducting a searching analysis of the feminine form. Rembrandt was an extremely innovative printmaker who experimented continuously. He understood that in etching he could obtain varied effects by controlling the inking and wiping of the plate and by printing on different types of paper. For this rare, early impression, Rembrandt carefully wiped the plate clean so that a thin layer of tone would unify the work and create the illusion that the figure is emerging from a shadowy background. Its beauty is further enhanced by the beige-toned Japanese paper, which adds warmth and a special glow to the flesh. Understandably, this superb impression once belonged to the Dukes of Devonshire and Chatsworth, who formed one of the finest old master print collections still in private hands.</td>
      <td>https://clevelandart.org/art/1997.4</td>
      <td><img src="https://piction.clevelandart.org/cma/ump.di?e=01C5ADF2D598CDDED58600436F54640BDD0A09E3BF47DB6DE8E2AC6CF16F9826&s=21&se=83547869&v=4&f=1997.4_o10.jpg" style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>435</th>
      <td>Manjusuri and Sea Turtle</td>
      <td>Mayumi Oda</td>
      <td>1989</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>The feminist artist Mayumi Oda portrays Manjusuri, the traditionally male bodhisattva of perfect wisdom, as a voluptuous nude woman. Twentieth-century Japanese artists present such traditional subjects in new ways and use modern printmaking techniques. Like ukiyo-e prints, black outlines\r\ndelineate flat forms here, but Oda employed the 20th-century medium of screenprinting.</td>
      <td>https://clevelandart.org/art/1994.77.a</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>436</th>
      <td>Manjusuri and Sea Turtle</td>
      <td>Mayumi Oda</td>
      <td>1989</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>The feminist artist Mayumi Oda portrays Manjusuri, the traditionally male bodhisattva of perfect wisdom, as a voluptuous nude woman. Twentieth-century Japanese artists present such traditional subjects in new ways and use modern printmaking techniques. Like ukiyo-e prints, black outlines\r\ndelineate flat forms here, but Oda employed the 20th-century medium of screenprinting.</td>
      <td>https://clevelandart.org/art/1994.77.b</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>437</th>
      <td>Manjusuri and Sea Turtle (diptych)</td>
      <td>Mayumi Oda</td>
      <td>1989</td>
      <td>Japan</td>
      <td>None</td>
      <td>None</td>
      <td>The feminist artist Mayumi Oda portrays Manjusuri, the traditionally male bodhisattva of perfect wisdom, as a voluptuous nude woman. Twentieth-century Japanese artists present such traditional subjects in new ways and use modern printmaking techniques. Like ukiyo-e prints, black outlines\r\ndelineate flat forms here, but Oda employed the 20th-century medium of screenprinting.</td>
      <td>https://clevelandart.org/art/1994.77</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
    <tr>
      <th>438</th>
      <td>Portrait of a Lady with an Elaborate Cartouche</td>
      <td>Giuseppe Cades (Italian, 1750-1799)</td>
      <td>1785</td>
      <td>Italy</td>
      <td>None</td>
      <td>None</td>
      <td>The woman portrayed in the oval has not yet been identified, but she is made to look highly important by her rendition in profile, like an ancient Renaissance coin, and by the elaborate decoration surrounding the cartouche (the central area containing the portrait). Increasing her distinction, the flying male nudes personify Victory and Fame, while the crown indicates her nobility. The contrast in technique between the cartouche and the portrait was purposefully emphasized to suggest that the portrait is an older "icon" of reverence. The finished quality of the drawing suggests that it was a presentation sheet and not a study for a work of art in another medium. Giuseppe Cades was an important history painter and frescoist, known also for his decorative ensembles. He worked mostly in Rome in a neoclassical style based on Renaissance and ancient Roman art. This drawing combines the contemporary rococo style of the red chalk portrait with the antique-based style of the surrounding decorative figures and forms.</td>
      <td>https://clevelandart.org/art/1999.172</td>
      <td><img src="Sorry! No URL available." style=max-width:250px;"/></td>
    </tr>
  </tbody>
</table>




```python

```
