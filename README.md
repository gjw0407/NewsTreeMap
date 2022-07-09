# NewsTreeMap

## Visualize News Topics

Most newspaper companies show headlines in raw text, but sometimes readers want to get a glimpse of current issues. 
This project crawls healines from various online newspaper stand and creates a treemap with keywords that best describe the headlines. 


**Crawling**
- Collects article from portal sites (daum, naver)

#### Article on daum.net (2022-07-09):
![image](https://user-images.githubusercontent.com/58447982/178089521-956512d3-840f-4860-a507-5eb1a062870c.png)


#### Extracted Article:

![image](https://user-images.githubusercontent.com/58447982/178089510-f5f52ea4-2bb1-42c6-b830-8cd92a00c5f7.png)



```c
def get_articles(self, url):
    print("Extracting Articles from", url)
    webpage = session.get(url)
    soup = BeautifulSoup(webpage.content, "html.parser")

    with open(os.path.join(os.path.dirname(__file__), "../memo.txt"), 'w', encoding='UTF-8') as fw:
        fw.write(str(requests.get(url).text))
        fw.close()

    cnt = 0
    for art in soup.find_all("strong", "tit_g"):
        articleURL = re.search("(?P<url>https?://[^\s]+)", str(art)).group("url")
        articleTitle = art.text
        self.URLBox.append(articleURL)
        self.titleBox.append(preprocess(articleTitle))
        # TODO
        # ADD Article Media (<span class="txt_info">이데일리</span>)
        cnt += 1
```

**NLP**
- Process NLP and form cluster
1. Standarize words that are used interchangably (eg. US/USA/America/The States --> US)
```c
def replace_politics(sentence):
politics_chinese_char_dict = {
                     "與" : "여당",
                     "尹" : "윤석열",
                     "野" : "야당"
                    }
for chinese_char, politics_name in politics_chinese_char_dict.items():
  sentence = sentence.replace(chinese_char, politics_name+" ")
return sentence  
```

2. Generate tokens and similarity map
```c
vectorizer = CountVectorizer() # HashingVectorizer(n_features=10000, stop_words=None)
title_noun_list = list(map(" ".join, articles['title_noun'].tolist()))
vectorizer.fit(title_noun_list)
title_noun_vector= vectorizer.transform(title_noun_list).toarray()
# articles["title_noun_vector"]= pd.Series(title_noun_vector)

similarity = [[0]*title_noun_vector.shape[0] for _ in range(title_noun_vector.shape[0])]

for i in range(title_noun_vector.shape[0]):
  for j in range(title_noun_vector.shape[0]):
    cos_sim = cosine_similarity(title_noun_vector[i:i+1],title_noun_vector[j:j+1])
    similarity[i][j] = float(cos_sim[0])
    similarity[j][i] = float(cos_sim[0])
```

3. Generate Clusters (check nlp.py for more detail)
```c
cls = News_clustering(similarity, 0.1, 1, labels) # similarity, 개수
new_labels = cls.cluster()
```

**Django**
- Display clusters as a treepmap
- Django detects changes on the website, and update new article to the treemap

Check Update/Save Changes (main.py)
```c
@timeloop.job(interval=timedelta(seconds=600))
def sample_job_every_2s():
    url = [read_csv()]
    news = NewsStand()
    news.checkChanges(url)
    saveArticle(news)
```

**Conclusion**
- TODO: More data (Maybe from different sources?)

![image](https://user-images.githubusercontent.com/58447982/175814144-616d05b5-8df2-443d-8249-e60afbfd6612.png)
