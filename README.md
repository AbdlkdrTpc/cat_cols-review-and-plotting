# cat_cols-review-and-plotting
Büyük veri yapılarında, Kategorik değişkenleri bulma, inceleme ve grafik çizdirme işlemi


import numpy as np
import pandas as pd
import seaborn as sns
pd.set_option('display.width', 500)
pd.set_option('display.max_columns', None)

df = sns.load_dataset("titanic")

df.columns
df.info()

# 1) tipi kategorik olan değişkenleri yakalayarak cat_cols adındaki değişkene list comp. yöntemi ile atayalım.
cat_cols = [i for i in df.columns if str(df[i].dtypes) in ["category", "object", "bool"]]

# 2) tipi sayısal olan ve bağımsız değişken sayısı 10 dan küçük olan değişkenlerin kategorik değişken olabileceği varsayımı ile bu değişkenleri yakalayarak num_bat_cat adındaki değişkene list comp. yöntemi ile atayalım.
num_but_cat = [i for i in df.columns if df[i].dtypes in ["int", "float"] and df[i].nunique() < 10]

# 1) ve 2) den ortaya çıkan listeleri toplayıp cat_cols listesine tekrardan atayalım.
cat_cols = cat_cols + num_but_cat

# 3) Kategorik değişkenler arasında kardinalitesi yüksek olan çok bir anlam ifade etmeyecek kategorik değişkenleri de bulalım.
car_but_cat = [i for i in df.columns if str(df[i].dtypes) in ["category", "object"] and df[i].nunique() > 20]

# 3) de bulduğumuz kardinalitesi yüksek kategorik değişkenler, cat_cols listesinde var ise onları çıkaralım.
cat_cols = [i for i in cat_cols if i not in car_but_cat]


df[cat_cols].nunique()
# Numerik değişkenleri de, df.columns içerisinde ki değişkenlerde gezip cat_cols içerisinde olmayanları seçerek oluşturalım.
num_cols = [i for i in df.columns if i not in cat_cols]


def cat_and_num_summary(dataframe):
    """
    Belirli kriterlere dayalı olarak bir DataFrame içindeki sütunları kategorik ve sayısal tiplere ayırır.

    Parameters
    ----------
    dataframe: pandas.DataFrame
        Analiz edilecek olan DataFrame

    Returns
    ---------
    cat_cols : list
        Kategorik olarak kabul edilen sütun adlarının bir listesi. Bu, sütunları içerir
        'category', 'object' veya 'bool' veri türlerine sahip sütunlar, ayrıca benzersiz değeri
        10'dan az olan sayısal sütunları içerir.

    num_cols : list
        Sayısal olarak kabul edilen sütun adlarının bir listesi. Bunlar, kategorik olarak sınıflandırılmak için
        belirtilen kriterleri karşılamayan sayısal sütunlardır.

    Notes
    --------
    - Kategorik sütunlar yüksek kardinaliteye sahipse (20'den fazla benzersiz değer), 'cat_cols' dışında bırakılır.
    - İşlev, veri türleri ve benzersiz değer sayısını kullanarak sınıflandırma kararları verir.

    """
    for i in dataframe.columns:
        # 1) tipi kategorik olan değişkenleri yakalayarak cat_cols adındaki değişkene list comp. yöntemi ile atayalım.
        cat_cols = [i for i in dataframe.columns if str(dataframe[i].dtypes) in ["category", "object", "bool"]]
        # 2) tipi sayısal olan ve bağımsız değişken sayısı 10 dan küçük olan değişkenlerin kategorik değişken olabileceği varsayımı ile bu değişkenleri yakalayarak num_bat_cat adındaki değişkene list comp. yöntemi ile atayalım.
        num_but_cat = [i for i in dataframe.columns if dataframe[i].dtypes in ["int", "float"] and dataframe[i].nunique() < 10]
        # 1) ve 2) den ortaya çıkan listeleri toplayıp cat_cols listesine tekrardan atayalım.
        cat_cols = cat_cols + num_but_cat
        # 3) Kategorik değişkenler arasında kardinalitesi yüksek olan çok bir anlam ifade etmeyecek kategorik değişkenleri de bulalım.
        car_but_cat = [i for i in dataframe.columns if str(dataframe[i].dtypes) in ["category", "object"] and dataframe[i].nunique() > 20]
        # 3) de bulduğumuz kardinalitesi yüksek kategorik değişkenler, cat_cols listesinde var ise onları çıkaralım.
        cat_cols = [i for i in cat_cols if i not in car_but_cat]
        # Numerik değişkenleri de, df.columns içerisinde ki değişkenlerde gezip cat_cols içerisinde olmayanları seçerek oluşturalım.
        num_cols = [i for i in df.columns if i not in cat_cols]
    return cat_cols, num_cols
cat_cols, num_cols = cat_and_num_summary(df)

import matplotlib.pyplot as plt


def cat_summary(dataframe, col, plot=False):
    """
    Belirtilen sütunun kategorik veri özetini yazdırır ve isteğe bağlı olarak bir grafik çizer.

    Parameters:
    -----------
    dataframe : pandas.DataFrame
        Analiz edilecek DataFrame.

    col : str
        Özetlenmek istenen sütunun adı.

    plot : bool, optional (default=False)
        Eğer True ise, sütunun dağılımını gösteren bir sütun grafiği çizer.

    Returns:
    --------
    None

    Notes:
    ------
    - Kategorik veri özeti, sütundaki her benzersiz değeri ve bu değerlere karşılık gelen gözlemlerin yüzde oranını içerir.
    - Eğer 'plot' True olarak ayarlanırsa, bir sütun grafiği çizilir.

    """
    print("################### " + col + " ###################")
    print(pd.DataFrame({col: dataframe[col].value_counts(),
                        "% oran": dataframe[col].value_counts()*100/len(dataframe[col])}))
    if plot:
        sns.countplot(x=col, data=dataframe)
        plt.show(block=True)


for i in cat_cols:
    cat_summary(df, i, plot=True)
