```python
#!pip install cufflinks
```


```python
import numpy as np
import pandas as pd
from sklearn import datasets
import ipywidgets as widgets
from ipywidgets import interact, interact_manual
import cufflinks as cf
cf.go_offline(connected=True)
```


<script type="text/javascript">
window.PlotlyConfig = {MathJaxConfig: 'local'};
if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
if (typeof require !== 'undefined') {
require.undef("plotly");
requirejs.config({
    paths: {
        'plotly': ['https://cdn.plot.ly/plotly-latest.min']
    }
});
require(['plotly'], function(Plotly) {
    window._Plotly = Plotly;
});
}
</script>




```python
#!pip install ipywidgets
```


```python
!jupyter nbextension enable --py widgetsnbextension
```

    Enabling notebook extension jupyter-js-widgets/extension...
          - Validating: ok
    


```python
data=pd.read_csv('태그클러스터.csv')
```


```python
@interact
def show_data_more_than(column=['category'], 
                        x=(0,7,1)):
    return data.loc[data[column] == x]
```


    interactive(children=(Dropdown(description='column', options=('category',), value='category'), IntSlider(value…



```python
a = ['걷기/등산', '휴식/힐링', '액티비티', '쇼핑', '체험관광', '역사']# 여행의 목적
b = ['안개/흐림', '비/눈', '맑음', '흐림']# 날씨
c = ['청초밭', '삼나무길',  '섬속의섬', '해안', '산', '오름', '동굴', '꽃', '숲길', '자연', '계곡', '한라산', '바다', '절벽', '용암', '모래', '해양', '포구', '연못', '지질', '분화구', '폭포', '바위', '초원', '바람']# 자연
d = ['중/장년', '청년', '노년', '아이']# 연령대
e = ['함께', '친구', '부모', '어머니', '혼자', '커플']# 동반상태
f = ['여름', '봄', '겨울', '가을', '사계절']# 계절# 이거순서
g = ['4.3', '경관/포토', '식물', '예술', '문화', '생태', '동물', '단풍', '분위기', '축제', '공연', '사진'] # 테마
h = ['미술/박물관', '전시/행사', '우수관광사업체', '해안', '산', '오름', '세계자연유산', '수목원', '파크', '사찰', '문화유적지', '광장', '공원', '테마공원', '휴양림', '야영장', '카페', '해수욕장', '전망대', '정원', '산책로', '해변', '포토스팟' ,'펍', '캠핑장', '등대', '목장']# 방문하고 싶은 곳
i = ['희생', '용천수', '작품', '건물', '테마', '작다', '관광', '우유',  '해녀', '우주', '올레', '조랑말', '마을'] # 기타
j = ['밤', '일출', '일몰'] # 시간대
k = ['자전거', '걷기', '산책', '승마', '물놀이', '드라이브', '골프']# 하고싶은 것
l = ['유채꽃',  '녹차', '감귤', '코스모스', '녹차밭', '동백나무', '수국'] #특산물

```


```python

```


```python
a =widgets.RadioButtons(options=['걷기/등산', '휴식/힐링', '액티비티', '쇼핑', '체험관광','상관없음'], value="상관없음")
b=widgets.RadioButtons(options=['안개/흐림', '비/눈', '맑음', '흐림','상관없음'], value="상관없음")
c=widgets.RadioButtons(options=['청초밭', '삼나무길',  '섬속의섬', '해안', '산', '오름', '동굴', '꽃', '숲길', '자연', '계곡', '한라산', '바다', '절벽', '용암', '모래', '해양', '포구', '연못', '지질', '분화구', '폭포', '바위', '초원', '바람','상관없음'], value="상관없음")
d=widgets.RadioButtons(options=['함께', '친구', '부모', '어머니', '혼자', '커플','상관없음'], value="상관없음")
e=widgets.RadioButtons(options=['여름', '봄', '겨울', '가을', '사계절','상관없음'], value="상관없음")
f= widgets.RadioButtons(options=['4.3', '경관/포토', '식물', '예술', '역사', '문화', '생태', '동물', '단풍', '분위기', '축제', '공연', '사진','상관없음'], value="상관없음")
g= widgets.RadioButtons(options=['미술/박물관', '전시/행사', '우수관광사업체', '해안', '산', '오름', '세계자연유산', '수목원', '파크', '사찰', '문화유적지', '광장', '공원' '테마공원', '휴양림', '야영장', '카페', '해수욕장', '전망대', '정원', '산책로', '해변', '포토스팟' ,'펍', '캠핑장', '등대', '목장','상관없음'], value="상관없음")
h= widgets.RadioButtons(options=['희생', '용천수', '작품', '건물', '테마', '작다', '관광', '우유',  '해녀', '우주', '올레', '조랑말', '마을','상관없음'], value="상관없음")
i= widgets.RadioButtons(options=['밤', '일출', '일몰','상관없음'], value="상관없음")
j= widgets.RadioButtons(options=['자전거', '걷기', '산책', '승마', '물놀이', '드라이브', '골프','상관없음'], value="상관없음")
k=widgets.RadioButtons(options=['유채꽃',  '녹차', '감귤', '코스모스', '녹차밭', '동백나무', '수국','상관없음'], value="상관없음")
l=widgets.RadioButtons(options=['중/장년', '청년', '노년', '아이','상관없음'], value="상관없음")
accordion = widgets.Accordion(children=[a,b,c,d,e,f,g,h,i,j,k,l])
accordion.set_title(0, '여행의 목적')
accordion.set_title(1, '날씨')
accordion.set_title(2, '자연')
accordion.set_title(11, '연령대')
accordion.set_title(3, '동반상태')
accordion.set_title(4, '계절')
accordion.set_title(5, '테마')
accordion.set_title(6, '방문하고 싶은 곳')
accordion.set_title(7, '기타')
accordion.set_title(8, '시간대')
accordion.set_title(9, '하고싶은 것')
accordion.set_title(10, '특산물')


accordion
```


    Accordion(children=(RadioButtons(index=5, options=('걷기/등산', '휴식/힐링', '액티비티', '쇼핑', '체험관광', '상관없음'), value='상관없…


# 이전에 상관없음은 그냥 리스트에서 없애는 과정


```python
result=[a.value,b.value,c.value,d.value,e.value,f.value,g.value,h.value,i.value,j.value,k.value,l.value]
```


```python
def erase(result):
    result=list(set(result))
    result.remove('상관없음')
    return(result)
```


```python
erase(result)
```




    []


