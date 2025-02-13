# sddk

`sddk` is a Python package for writting and reading files to/from [sciencedata.dk](https://sciencedata.dk/). Since version 3.0, it also supports [owncloud.cesnet.cz](https://www.cesnet.cz/services/owncloud/?lang=en). In the future, it will support other  providers from the [CS3MESH4EOSC](https://cs3mesh4eosc.eu) inciative. It is especially designed for working with shared folders. It relies mainly upon Python requests library.

sciencedata.dk is a project managed by [DEiC](https://www.deic.dk) (Danish e-infrastrcture cooperation) aimed to offer a robust data storage, data management and data publication solution for researchers in Denmark and abroad (see [docs](https://sciencedata.dk/sites/user/) and [dev](https://sciencedata.dk/sites/developer/) for more info). The storage is accessible either through (1)  the web interface, (2) WebDAV clients, (3) OwnCloud/NextCloud desktop applications or (4) API relaying on the HTTP Protocol. One strength of sciencedata.dk is that it currently supports institutional login from 2976 research and educational institutions around the globe (using [WAYF](https://www.wayf.dk/en/about)). That makes it a perfect tool for international research collaboration. 

The main functionality of the package is in uploading any Python object (str, dict, list, dataframe or figure) as a file to a preselected personal or shared folder on the cloud platform and getting it back into Python as the original Python object. It uses sciencedata.dk API in combination with Python requests library.

## Install and import

To install the package within within command line, run:

```bash
pip install sddk # # to have the latest version, use flag "--ignore-installed"
```

To install tje package within Jupyter environment, run:

```python
!pip install sddk # to have the latest version, use flag "--ignore-installed"
```

Once installed, import the package in the following way:

```python
import sddk
```
## Authentification

To establish the cloud session, you have to know the following:

* name of the **service provider**; currently we support two options: `sciencedata.dk` or `owncloud.cesnet.cz` (`sciencedata.dk` by default)

* **username/ID** from the service provider (e.g. "123456@au.dk" or "1fcd40da27c3573f1479718227a43e1a5426aac1"),
* **password / token** from the provider (has to be previously configured or generated manually using the web interface of the provider),

In the case you want to access a shared folder, you further need:

* **name** of the shared folder you want to access (e.g. "our_shared_folder"),
* **username / id** of the owner of the folder (if it is not yours)

### cloudSession()

* parameters:
  * `provider` - default: "sciencedata.dk"; alternatively "owncloud.cesnet.cz"
  * `shared_folder_name` - name of the shared folder; default `None`
  * `owner` - username of the owner of the shared folder; default `None`

In the case of a shared folder, you might be either its owner, or it might be a folder which has been shared with you by someone else, who is its owner- one important feature of the package is that in both cases you use *exactly the same syntax*. That means all members of a team can configure the session and access the folder using the same piece of code, the rest is entered interactively.

Calling the `cloudSession()` class, you configure a an authorized session class object `s`, which supports an array of useful functions.

### Establish personal session

```python
s = cloudSession() # "sciencedata.dk by default for owncloud.cesnet.cz, run:
# s = cloudSession("owncloud.cesnet.cz")
```

### Establish session with root in shared folder

To configure a session pointing to a shared folder, run:

```python
s = sddk.cloudSession("sciencedata.dk", "our_shared_folder", "owner_username@au.dk")
```

Subsequently, you can locate your files in relative path to this root folder ("our_shared_folder") 

## write_file()

The most important components of the package are two functions: `write_file(path_and_filename, python_object)` and `read_file(path_and_filename, type_of_object)`. 

So far these functions can be used with several different types of Python objects: `str`, `list`, `dictionary`, pandas' `dataframe`, geopandas `geodataframe` , matplotlib's `figure`, and plotly image object. These can be written either as `.txt`, `.json`, `geojson` , `.png` or `.eps` files, based upon type of the input object and filename's ending chosen by the user. Here are simple instances of these python objects to play with:

```python
### Python "str" object
string_object =  "string content"
### Python "list" object
list_object = ['a', 'b', 'c', 'd']
### Python "dictionary" object
dict_object = {"a" : 1, "b" : 2, "c":3 }
### Pandas dataframe object
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"]) 
### Matplotlib figure object
import matplotlib.pyplot as plt
figure_object = plt.figure() # generate object
plt.plot(range(10)) # fill it by plotted values
### (the same also works for plotly figures)
```

The simplest example is once we want to write a string object into a textfile located at our home folder (or shared folder)

```python
s.write_file("test_string.txt", string_object)
```

In the case  that everything is fine, you will receive following message:

```
> Your <class 'str'> object has been succefully written as "https://sciencedata.dk/files/test_string.txt"
```

However, there is a couple of things which might go wrong - You can choose an unsupported python object, a non-existent path or unsupported file format. The function captures some of these cases. For instance, once you run `sddk.write_file("nonexistent_folder/filename.wtf", string_object, conf)`, you will be interactively asked for corrections. First: the function checks whether the path is correct. When corrected to an existent folder (here it is "personal_folder"), the function further inspect whether it has known ending (i.e. `txt`, `json`, `feather`, or `png`). If not, it asks you interactively for correction. Third, it checks whether the folder already contain a file of the same name (to avoid unintended overwritting), and if yes, asks you what to do. Finally, it prints out where you can find your file and what type of object it encapsulates. 

```
>>> The path is not valid. Try different path and filename: textfile.wtf
>>> Unsupported file format. Type either "txt", "json", or "png"
>>> A file with the same name ("textfile.txt") already exists in this location.
Press Enter to overwrite it or choose different path and filename: textfile2.txt
```

The same function works with dictionaries, lists, Matplotlib's figures and especially Pandas' dataframes. Pandas' dataframe is our favorite. We send there and back 1GB+ dataframes as json or feather files on a daily basis. See examples below.

## read_file()

On the other side, we have the function `s.read_file(path_and_filename, object_type)`, which enables us to to read our files back to python as chosen python objects. Currently, the function can read  textfiles as strings, json files as either dictionary, lists or Pandas's dataframes, and geojson files as geopandas GeoDataFrames. You have to specify the type of object as the second argument, the values are either "str", "list", "dict", "df" or "gdf" within quotation marks, like in these examples. If you omit this, the file is parsed as pandas DataFrame.

```python
string_object = read_file("test_string.txt", "str")
string_object
>>> 'string content'
```

```python
list_object = read_file("simple_list.json", "list")
list_object
>>> ['a', 'b', 'c', 'd']
```

```python
dict_object = read_file("simple_dict.json", "dict")
dict_object
>>> {'a': 1, 'b': 2, 'c': 3}
```

```python
dataframe_object = read_file("simple_df.json")
dataframe_object
>>>     a   b   c
0  a1  b1  c1
1  a2  b2  c2
```

## Examples

### pandas.DataFrame to `.json` and back


```python
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"])
dataframe_object
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>



```python
s.write_file("simple_dataframe.json", dataframe_object)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.json"
```

```python
# read the file back as a new object "df_back"
df_back = s.read_file("simple_dataframe.json")
df_back
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>


Reading a larger dataframe file from a public folder:


```python
%%time
EDH_sample = s.read_file("https://sciencedata.dk/public/8fe7d59de1eafe5f8eaebc0044534606/EDH_sample.json")
EDH_sample.head(5)
# alternatively, you can use it by setting the three arguments (it is just a matter of taste):
# EDH_sample = sddk.read_file("EDH_sample.json", "df", public_folder="8fe7d59de1eafe5f8eaebc0044534606")
EDH_sample.head(5)
# this is an example usage of public folder, see below for explanation.
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>diplomatic_text</th>
      <th>literature</th>
      <th>trismegistos_uri</th>
      <th>id</th>
      <th>findspot_ancient</th>
      <th>not_before</th>
      <th>type_of_inscription</th>
      <th>work_status</th>
      <th>edh_geography_uri</th>
      <th>not_after</th>
      <th>...</th>
      <th>external_image_uris</th>
      <th>religion</th>
      <th>fotos</th>
      <th>geography</th>
      <th>military</th>
      <th>social_economic_legal_history</th>
      <th>coordinates</th>
      <th>text_cleaned</th>
      <th>origdate_text</th>
      <th>objecttype</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>D M / NONIAE P F OPTATAE / ET C IVLIO ARTEMONI...</td>
      <td>AE 1983, 0192.; M. Annecchino, Puteoli 4/5, 19...</td>
      <td>https://www.trismegistos.org/text/251193</td>
      <td>HD000001</td>
      <td>Cumae, bei</td>
      <td>0071</td>
      <td>epitaph</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0130</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>40.8471577,14.0550756</td>
      <td>Dis Manibus Noniae Publi filiae Optatae et Cai...</td>
      <td>71 AD – 130 AD</td>
      <td>[Tafel, 257]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>C SEXTIVS PARIS / QVI VIXIT / ANNIS LXX</td>
      <td>AE 1983, 0080. (A); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265631</td>
      <td>HD000002</td>
      <td>Roma</td>
      <td>0051</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>Caius Sextius Paris qui vixit annis LXX       ...</td>
      <td>51 AD – 200 AD</td>
      <td>[Tafel, 257]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[ ]VMMIO [ ] / [ ]ISENNA[ ] / [ ] XV[ ] / [ ] / [</td>
      <td>AE 1983, 0518. (B); J. González, ZPE 52, 1983,...</td>
      <td>https://www.trismegistos.org/text/220675</td>
      <td>HD000003</td>
      <td>None</td>
      <td>0131</td>
      <td>honorific inscription</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0170</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.37281,-6.04589</td>
      <td>Publio Mummio Publi filio Galeria Sisennae Rut...</td>
      <td>131 AD – 170 AD</td>
      <td>[Statuenbasis, 57]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[ ]AVS[ ]LLA / M PORCI NIGRI SER / DOMINAE VEN...</td>
      <td>AE 1983, 0533. (B); A.U. Stylow, Gerión 1, 198...</td>
      <td>https://www.trismegistos.org/text/222102</td>
      <td>HD000004</td>
      <td>Ipolcobulcula</td>
      <td>0151</td>
      <td>votive inscription</td>
      <td>checked with photo</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>[http://cil-old.bbaw.de/test06/bilder/datenban...</td>
      <td>names of pagan deities</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.4442,-4.27471</td>
      <td>AVSLLA Marci Porci Nigri serva dominae Veneri ...</td>
      <td>151 AD – 200 AD</td>
      <td>[Altar, 29]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[ ] L SVCCESSVS / [ ] L L IRENAEVS / [ ] C L T...</td>
      <td>AE 1983, 0078. (B); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265629</td>
      <td>HD000005</td>
      <td>Roma</td>
      <td>0001</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>libertus Successus  Luci libertus Irenaeus  C...</td>
      <td>1 AD – 200 AD</td>
      <td>[Stele, 250]</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>



### pandas.DataFrame to `.feather` and back

This might cause issues because of the way how pandas implements pyarrow and feather. To work with feather, check that you have installed a correct version of `pyarrow` package:

```python
import pyarrow
pyarrow.__version__
```

You need 0.17.1 or higher.  Google colab comes with 0.14.1 by default, so you have to upgrade:

```python
!pip install pyarrow --upgrade
```

and restart your runtime.

Originally,  sddk 1.9-2.4 specified the requirement `pyarrow>=0.17.1` , but it produced a lot of conflicts during an installation on Google colab, since there many other packages requiring pyarrow==0.14.1. Therefore, pyarrow is currently bypassed.


```python
s.write_file("simple_dataframe.feather", dataframe_object)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.feather"
```

```python
s.read_file("simple_dataframe.feather")
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>




Reading a larger file from public folder


```python
%%time
EDH_sample = s.read_file("https://sciencedata.dk/public/8fe7d59de1eafe5f8eaebc0044534606/EDH_sample.feather")
EDH_sample.head(5)
# alternative solution:
# EDH_sample = s.read_file("EDH_sample.feather", "df", "8fe7d59de1eafe5f8eaebc0044534606")
EDH_sample.head(5)
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>diplomatic_text</th>
      <th>literature</th>
      <th>trismegistos_uri</th>
      <th>id</th>
      <th>findspot_ancient</th>
      <th>not_before</th>
      <th>type_of_inscription</th>
      <th>work_status</th>
      <th>edh_geography_uri</th>
      <th>not_after</th>
      <th>...</th>
      <th>external_image_uris</th>
      <th>religion</th>
      <th>fotos</th>
      <th>geography</th>
      <th>military</th>
      <th>social_economic_legal_history</th>
      <th>coordinates</th>
      <th>text_cleaned</th>
      <th>origdate_text</th>
      <th>objecttype</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>D M / NONIAE P F OPTATAE / ET C IVLIO ARTEMONI...</td>
      <td>AE 1983, 0192.; M. Annecchino, Puteoli 4/5, 19...</td>
      <td>https://www.trismegistos.org/text/251193</td>
      <td>HD000001</td>
      <td>Cumae, bei</td>
      <td>0071</td>
      <td>epitaph</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0130</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>40.8471577,14.0550756</td>
      <td>Dis Manibus Noniae Publi filiae Optatae et Cai...</td>
      <td>71 AD – 130 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>C SEXTIVS PARIS / QVI VIXIT / ANNIS LXX</td>
      <td>AE 1983, 0080. (A); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265631</td>
      <td>HD000002</td>
      <td>Roma</td>
      <td>0051</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>Caius Sextius Paris qui vixit annis LXX       ...</td>
      <td>51 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[ ]VMMIO [ ] / [ ]ISENNA[ ] / [ ] XV[ ] / [ ] / [</td>
      <td>AE 1983, 0518. (B); J. González, ZPE 52, 1983,...</td>
      <td>https://www.trismegistos.org/text/220675</td>
      <td>HD000003</td>
      <td>None</td>
      <td>0131</td>
      <td>honorific inscription</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0170</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.37281,-6.04589</td>
      <td>Publio Mummio Publi filio Galeria Sisennae Rut...</td>
      <td>131 AD – 170 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[ ]AVS[ ]LLA / M PORCI NIGRI SER / DOMINAE VEN...</td>
      <td>AE 1983, 0533. (B); A.U. Stylow, Gerión 1, 198...</td>
      <td>https://www.trismegistos.org/text/222102</td>
      <td>HD000004</td>
      <td>Ipolcobulcula</td>
      <td>0151</td>
      <td>votive inscription</td>
      <td>checked with photo</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>names of pagan deities</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.4442,-4.27471</td>
      <td>AVSLLA Marci Porci Nigri serva dominae Veneri ...</td>
      <td>151 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[ ] L SVCCESSVS / [ ] L L IRENAEVS / [ ] C L T...</td>
      <td>AE 1983, 0078. (B); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265629</td>
      <td>HD000005</td>
      <td>Roma</td>
      <td>0001</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>libertus Successus  Luci libertus Irenaeus  C...</td>
      <td>1 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>

### pandas.DataFrame to `.csv` and back 


```python
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"]) 
dataframe_object
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>





```python
sddk.write_file("simple_dataframe.csv", dataframe_object)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.csv"
```

```python
sddk.read_file("simple_dataframe.csv")
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>


## list_filenames()

This function enables you to list all files within a directory. You can specify the directory, type of the file you are interested in and the conf variable. For instance, the function belows returns all JSON files within your main directory.

```python
 s.list_filenames(filetype="json")
```

## Personal, shared and public folders

**Shared in and out**

One of the main strength of the sciencedata.dk are collaborative features, namely the way you can manage its **shared** and **public** folders.

**Shared** folders always have one of two forms: either (1) a shared folder *you* share with some users or (2) a shared folder someone else shares with you. 

Each shared folder has its **owner**. The folders  are located in their owner's  personal space and can be easily accessed  from there like from any other personal folder. However, in the case of shared folders you do not own (i.e. which were shared with you by someone else) you also need to know the username of their owner. 

One of the key features of the **sddk** package is that it enables you to access both types of shared folders **using exactly the same syntax**, regardless you are their owner or not. This enables that all members of a team accessing a folder owned and shared by one member can you use the same code. The function just checks both options and chooses what works.

For instance, a project member with username `member1@inst.org` created a folder in his personal space called `team_folder`,  uploaded there a file called `textfile.txt`, and shared the folder with his teammates with usernames `member2@inst.org` and `member3@inst.org`. All of them can now access the file using the same series of commands:

**Public files and folders**

Sciencedata.dk also enables to produce public files and folders. These files and folders might be accessed using `sddk.read_file()` function even without having sciencedata.dk account. You just have to know  share link code of the file or folder. To read a public file, you can use:

```python
public_file_code = "3e0a55a4182de313e04523360cecd015"
gospels_cleaned = s.read_file("https://sciencedata.dk/public/" + public_file_code, "dict")
# of course, you can write it directly:
# gospels_cleaned = s.read_file("https://sciencedata.dk/public/3e0a55a4182de313e04523360cecd015", "dict")
```

Public files can be read even if you are not logged into a session at the moment (using `sddk.read_file()` instead of `s.read_file()`)

```python
gospels_cleaned = sddk.read_file("https://sciencedata.dk/public/" + public_file_code, "dict")
```

To read a specific file within a public folder, you can use the code below, i.e. you can replace the `conf` parameter by sharing code of the public folder.

```python
public_folder_code = "31b393e2afe1ee96ce81869c7efe18cb"
c_aristotelicum = sddk.read_file("c_aristotelicum.json", "df", public_folder_code)
```

## write_file()

The most important components of the package are two functions: `write_file(path_and_filename, python_object, conf)` and `read_file(path_and_filename, type_of_object, conf)`. 

So far these functions can be used with several different types of Python objects: `str`, `list`, `dictionary`, pandas' `dataframe`, geopandas `geodataframe` and matplotlib's `figure`. These can be written either as `.txt`, `.json`, `geojson` , `.png` or `.eps` files, based upin type of the input object a d filename's ending chosen by the user. Here are simple instances of these python objects to play with:

```python
### Python "str" object
string_object =  "string content"
### Python "list" object
list_object = ['a', 'b', 'c', 'd']
### Python "dictionary" object
dict_object = {"a" : 1, "b" : 2, "c":3 }
### Pandas dataframe object
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"]) 
### Matplotlib figure object
import matplotlib.pyplot as plt
figure_object = plt.figure() # generate object
plt.plot(range(10)) # fill it by plotted values
### (the same also works for plotly figures)
```

The simplest example is once we want to write a string object into a textfile located at our home folder (Remember, that since the configuration this home folder is contained within the `sddk_url` variable ) 

```python
sddk.write_file("test_string.txt", string_object, conf)
```

In the case  that everything is fine, you will receive following message:

```
> Your <class 'str'> object has been succefully written as "https://sciencedata.dk/files/test_string.txt"
```

However, there is a couple of things which might go wrong - You can choose an unsupported python object, a non-existent path or unsupported file format. The function captures some of these cases. For instance, once you run `sddk.write_file("nonexistent_folder/filename.wtf", string_object, conf)`, you will be interactively asked for corrections. First: the function checks whether the path is correct. When corrected to an existent folder (here it is "personal_folder"), the function further inspect whether it has known ending (i.e. `txt`, `json`, `feather`, or `png`). If not, it asks you interactively for correction. Third, it checks whether the folder already contain a file of the same name (to avoid unintended overwritting), and if yes, asks you what to do. Finally, it prints out where you can find your file and what type of object it encapsulates. 

```
>>> The path is not valid. Try different path and filename: textfile.wtf
>>> Unsupported file format. Type either "txt", "json", or "png": txt
>>> A file with the same name ("textfile.txt") already exists in this location.
Press Enter to overwrite it or choose different path and filename: textfile2.txt
>>> Your <class 'str'> object has been succefully written as "https://sciencedata.dk/files/textfile2.txt"
```

The same function works with dictionaries, lists, Matplotlib's figures and especially Pandas' dataframes. Pandas' dataframe is my favorite. I send there and back 1GB+ dataframes as json or feather files on a daily basis. See examples below

## read_file()

On the other side, we have the function `sddk.read_file(path_and_filename, object_type)`, which enables us to to read our files back to python as chosen python objects. Currently, the function can read only textfiles as strings, and json files as either dictionary, lists or Pandas's dataframes. You have to specify the type of object as the second argument, the values are either "str", "list", "dict", "df" or "gdf" within quotation marks, like in these examples:

```python
string_object = read_file("test_string.txt", "str", conf)
string_object
>>> 'string content'
```

```python
list_object = read_file("simple_list.json", "list", conf)
list_object
>>> ['a', 'b', 'c', 'd']
```

```python
dict_object = read_file("simple_dict.json", "dict", conf)
dict_object
>>> {'a': 1, 'b': 2, 'c': 3}
```

```python
dataframe_object = read_file("simple_df.json", "df", conf)
>>>     a   b   c
0  a1  b1  c1
1  a2  b2  c2
```

## Examples

### pandas.DataFrame to `.json` and back


```python
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"])
dataframe_object
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>




```python
sddk.write_file("simple_dataframe.json", dataframe_object, conf)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.json"
```

```python
sddk.read_file("simple_dataframe.json", "df", conf)
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>

Reading a larger file from a public folder


```python
%%time
EDH_sample = sddk.read_file("EDH_sample.json", "df", "8fe7d59de1eafe5f8eaebc0044534606")
EDH_sample.head(5)
# this is an example usage of public folder, see below for explanation.
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>diplomatic_text</th>
      <th>literature</th>
      <th>trismegistos_uri</th>
      <th>id</th>
      <th>findspot_ancient</th>
      <th>not_before</th>
      <th>type_of_inscription</th>
      <th>work_status</th>
      <th>edh_geography_uri</th>
      <th>not_after</th>
      <th>...</th>
      <th>external_image_uris</th>
      <th>religion</th>
      <th>fotos</th>
      <th>geography</th>
      <th>military</th>
      <th>social_economic_legal_history</th>
      <th>coordinates</th>
      <th>text_cleaned</th>
      <th>origdate_text</th>
      <th>objecttype</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>D M / NONIAE P F OPTATAE / ET C IVLIO ARTEMONI...</td>
      <td>AE 1983, 0192.; M. Annecchino, Puteoli 4/5, 19...</td>
      <td>https://www.trismegistos.org/text/251193</td>
      <td>HD000001</td>
      <td>Cumae, bei</td>
      <td>0071</td>
      <td>epitaph</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0130</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>40.8471577,14.0550756</td>
      <td>Dis Manibus Noniae Publi filiae Optatae et Cai...</td>
      <td>71 AD – 130 AD</td>
      <td>[Tafel, 257]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>C SEXTIVS PARIS / QVI VIXIT / ANNIS LXX</td>
      <td>AE 1983, 0080. (A); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265631</td>
      <td>HD000002</td>
      <td>Roma</td>
      <td>0051</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>Caius Sextius Paris qui vixit annis LXX       ...</td>
      <td>51 AD – 200 AD</td>
      <td>[Tafel, 257]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[ ]VMMIO [ ] / [ ]ISENNA[ ] / [ ] XV[ ] / [ ] / [</td>
      <td>AE 1983, 0518. (B); J. González, ZPE 52, 1983,...</td>
      <td>https://www.trismegistos.org/text/220675</td>
      <td>HD000003</td>
      <td>None</td>
      <td>0131</td>
      <td>honorific inscription</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0170</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.37281,-6.04589</td>
      <td>Publio Mummio Publi filio Galeria Sisennae Rut...</td>
      <td>131 AD – 170 AD</td>
      <td>[Statuenbasis, 57]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[ ]AVS[ ]LLA / M PORCI NIGRI SER / DOMINAE VEN...</td>
      <td>AE 1983, 0533. (B); A.U. Stylow, Gerión 1, 198...</td>
      <td>https://www.trismegistos.org/text/222102</td>
      <td>HD000004</td>
      <td>Ipolcobulcula</td>
      <td>0151</td>
      <td>votive inscription</td>
      <td>checked with photo</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>[http://cil-old.bbaw.de/test06/bilder/datenban...</td>
      <td>names of pagan deities</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.4442,-4.27471</td>
      <td>AVSLLA Marci Porci Nigri serva dominae Veneri ...</td>
      <td>151 AD – 200 AD</td>
      <td>[Altar, 29]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[ ] L SVCCESSVS / [ ] L L IRENAEVS / [ ] C L T...</td>
      <td>AE 1983, 0078. (B); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265629</td>
      <td>HD000005</td>
      <td>Roma</td>
      <td>0001</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>libertus Successus  Luci libertus Irenaeus  C...</td>
      <td>1 AD – 200 AD</td>
      <td>[Stele, 250]</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>



### pandas.DataFrame to `.feather` and back

This might cause issues because of the way how pandas implements pyarrow and feather. To work with feather, check that you have installed a correct version of `pyarrow` package:

```python
import pyarrow
pyarrow.__version__
```

You need 0.17.1 or higher.  Google colab comes with 0.14.1 by default, so you have to upgrade:

```python
!pip install pyarrow --upgrade
```

and restart your runtime.

Originally,  sddk 1.9-2.4 specified the requirement `pyarrow>=0.17.1` , but it produced a lot of conflicts during an installation on Google colab, since there many other packages requiring pyarrow==0.14.1. Therefore, pyarrow is currently bypassed.


```python
sddk.write_file("simple_dataframe.feather", dataframe_object, conf)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.feather"
```

```python
sddk.read_file("simple_dataframe.feather", "df", conf)
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>



Reading a larger file from public folder


```python
%%time
EDH_sample = sddk.read_file("EDH_sample.feather", "df", "8fe7d59de1eafe5f8eaebc0044534606")
EDH_sample.head(5)
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>diplomatic_text</th>
      <th>literature</th>
      <th>trismegistos_uri</th>
      <th>id</th>
      <th>findspot_ancient</th>
      <th>not_before</th>
      <th>type_of_inscription</th>
      <th>work_status</th>
      <th>edh_geography_uri</th>
      <th>not_after</th>
      <th>...</th>
      <th>external_image_uris</th>
      <th>religion</th>
      <th>fotos</th>
      <th>geography</th>
      <th>military</th>
      <th>social_economic_legal_history</th>
      <th>coordinates</th>
      <th>text_cleaned</th>
      <th>origdate_text</th>
      <th>objecttype</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>D M / NONIAE P F OPTATAE / ET C IVLIO ARTEMONI...</td>
      <td>AE 1983, 0192.; M. Annecchino, Puteoli 4/5, 19...</td>
      <td>https://www.trismegistos.org/text/251193</td>
      <td>HD000001</td>
      <td>Cumae, bei</td>
      <td>0071</td>
      <td>epitaph</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0130</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>40.8471577,14.0550756</td>
      <td>Dis Manibus Noniae Publi filiae Optatae et Cai...</td>
      <td>71 AD – 130 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>C SEXTIVS PARIS / QVI VIXIT / ANNIS LXX</td>
      <td>AE 1983, 0080. (A); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265631</td>
      <td>HD000002</td>
      <td>Roma</td>
      <td>0051</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>Caius Sextius Paris qui vixit annis LXX       ...</td>
      <td>51 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[ ]VMMIO [ ] / [ ]ISENNA[ ] / [ ] XV[ ] / [ ] / [</td>
      <td>AE 1983, 0518. (B); J. González, ZPE 52, 1983,...</td>
      <td>https://www.trismegistos.org/text/220675</td>
      <td>HD000003</td>
      <td>None</td>
      <td>0131</td>
      <td>honorific inscription</td>
      <td>provisional</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0170</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.37281,-6.04589</td>
      <td>Publio Mummio Publi filio Galeria Sisennae Rut...</td>
      <td>131 AD – 170 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[ ]AVS[ ]LLA / M PORCI NIGRI SER / DOMINAE VEN...</td>
      <td>AE 1983, 0533. (B); A.U. Stylow, Gerión 1, 198...</td>
      <td>https://www.trismegistos.org/text/222102</td>
      <td>HD000004</td>
      <td>Ipolcobulcula</td>
      <td>0151</td>
      <td>votive inscription</td>
      <td>checked with photo</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>names of pagan deities</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>37.4442,-4.27471</td>
      <td>AVSLLA Marci Porci Nigri serva dominae Veneri ...</td>
      <td>151 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[ ] L SVCCESSVS / [ ] L L IRENAEVS / [ ] C L T...</td>
      <td>AE 1983, 0078. (B); A. Ferrua, RAL 36, 1981, 1...</td>
      <td>https://www.trismegistos.org/text/265629</td>
      <td>HD000005</td>
      <td>Roma</td>
      <td>0001</td>
      <td>epitaph</td>
      <td>no image</td>
      <td>https://edh-www.adw.uni-heidelberg.de/edh/geog...</td>
      <td>0200</td>
      <td>...</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>41.895466,12.482324</td>
      <td>libertus Successus  Luci libertus Irenaeus  C...</td>
      <td>1 AD – 200 AD</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>




```python
sddk.write_file("EDH_sample.feather", EDH_sample, conf)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/EDH_sample.feather"
```

### pandas.DataFrame to `.csv` and back 


```python
import pandas as pd
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"]) 
dataframe_object
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>




```python
sddk.write_file("simple_dataframe.csv", dataframe_object, conf)
> Your <class 'pandas.core.frame.DataFrame'> object has been succefully written as "https://sciencedata.dk/files/simple_dataframe.csv"
```

```python
sddk.read_file("simple_dataframe.csv", "df", conf)
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
    </tr>
  </tbody>
</table>

## list_filenames()

This function enables you to list all files within a directory. You can specify the directory, type of the file you are interested in and the conf variable. For instance, the function belows returns all JSON files within your main directory.

```python
 sddk.list_filenames(filetype="json", conf=conf)
```

## Personal, shared and public folders

**Shared in and out**

One of the main strength of the sciencedata.dk are collaborative features, namely the way you can manage its **shared** and **public** folders.

**Shared** folders always have one of two forms: either (1) a shared folder *you* share with some users or (2) a shared folder someone else shares with you. 

Each shared folder has its **owner**. The folders  are located in their owner's  personal space and can be easily accessed by them from there like any other personal folder.

However, in the case of shared folders you do not own (i.e. which were shared with you by someone else) you also need to know the username of their owner. 

One of the key features of the **sddk** package is that it enables you to access both types of shared folders **using exactly the same command**, regardless you are their owner or not. This enables that all members of a team accessing a folder owned and shared by one member can you use the same code. The function just checks both options and chooses what works.

For instance, a project member with username `member1@inst.org` created a folder in his personal space called `team_folder`,  uploaded there a file called `textfile.txt`, and shared the folder with his teammates with usernames `member2@inst.org` and `member3@inst.org`. All of them can now access the file using the same series of commands:

```python
# configure session with access to the shared folder:
conf = sddk.configure("team_folder", "member1@inst.org")
# read the file located in this shared folder:
sddk.read_file("testfile.txt", "str", conf)
```

**Public files and folders**

Sciencedata.dk also enables to produce public files and folders. These files and folders might be accessed using `sddk.read_file()` function even without having sciencedata.dk account. You just have to know  share link code of the file or folder. To read a public file, you can use:

```python
public_file_code = "3e0a55a4182de313e04523360cecd015"
gospels_cleaned = sddk.read_file("https://sciencedata.dk/public/" + public_file_code, "dict")
```

To read a specific file within a public folder, you can use the code below, i.e. you can replace the `conf` parameter by sharing code of the public folder.

```python
public_folder_code = "31b393e2afe1ee96ce81869c7efe18cb"
c_aristotelicum = sddk.read_file("c_aristotelicum.json", "df", public_folder_code)
```

## Credit

The package is continuously develepod and maintained by [Vojtěch Kaše](http://vojtechkase.cz) as a part of the digital collaborative research workflow of the [SDAM project](https://sdam-au.github.io/sdam-au/) at Aarhus University, Denmark. To cite this package, use:

## Version history

* 3.0 - new way of authentification, based on `cloudSession()` class object; it also supports `owncloud.cesnet.cz` as service provider
* 2.10 - supports `.geojson`
* 2.9 - `.eps` file format for matplotlib figures support (plotly works only with `.png`) 
* 2.8.2 - plotly support
* 2.7 - resolving issues #1 (reading public json files) & #2 (beautifulsoup import)
* 2.6 - pyarrow avoided
* 2.5 - pyarrow version changed back to unspecified
* 2.4 - json encoding bug removed
* 2.3 - json encoding
* 2.2 - setup.py update
* 2.1 - README.md update
* 2.0 - tested with `.txt`, `.json`, `.feather` and `.png`.
* 1.9 - supports public files and folders; supports `.feather` file format (`utf.8` enforced)
* 1.8 - `list_filenames()` function and `configure()` alias added
* 1.7 - figures
* 1.6.1 - bug
* 1.6 - enables writing dataframes as `csv`
* 1.5 - reads individually shared files without necessary configuration
* 1.4 - `json` package dependency
* 1.3 - `conf` corrected
* 1.2 - `conf` variable added
* 1.1 - a simple correction
* 1.0 -  functions `write_file()` and `read_file()` added
* 0.1.2 -  redirection added
* 0.1.1 - added shared folder owner argument to the main configuration function; migration from test.pypi to real pypi
* 0.0.8 - shared folders reading&writing for ordinary users finally functional
* 0.0.7 - configuration of individual session by default
* 0.0.6 - first functional configuration



