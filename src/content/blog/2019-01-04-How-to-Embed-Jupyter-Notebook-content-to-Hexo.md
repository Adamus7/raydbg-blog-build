---
title: Embed Jupyter Notebook into Hexo Post
pubDate: 2019-01-04 15:30:01
tags:
- Hexo
- Jupyter Notebook
---
Jupyter Notebook is a flexible tool that helps you create readable analyses which can keep code, images, comments, formulae and plots together. I also would like to make a post to discuss an data analyses result using markdown in my Hexo blog system. In this blog, I will share how to embed the content of Jupyter Notebook into Hexo using markdown format.
<!-- more -->
# Useful Ideas
There are several existing solutions on the Internet:
*   .ipynb -> .md -> Copy the content of .md into a Hexo post;
*   .ipynb -> .html -> Reference the HTML file in a Hexo post;
*   .ipynb -> .html -> Embed the HTML file in a iFrame;
*   .ipynb -> .pdf -> Embed the pdf file in a Hexo post using PDF tag.

However, I'm just not satisficed with any of them because of the inconsistent format and style.

# My Solution
During my testing, for solution 1, the only biggest problem is Hexo cannot render the DataFrame table very well as it is a raw HTML code in markdown file. Probably we can make it better.

## Convert .ipynb to .md
Run the following command to convert .ipynb file from Jupyter Notebook format to a markdown file.
```bash
jupyter nbconvert --to markdown  notebook.ipynb 
```
Two new items will be created after running the command:
* `notebook.md`: a new markdown file;
* `notebook_files`: a folder containing all the images in the notebook.

## Embed in a Hexo Post
Create a new post in Hexo and copy the content of notebook.md to the post body.
```
---
title: New Blog Post
---
<Copy the content to here>
```
Next, move the notebook_files folder to the resource folder of the post. Edit the image tag in Hexo post and point it to correct path.

### Fix the DataFrame Table Display Issue
You may notice that DataFrame table is transformed into HTLM format in the markdown file. It seems to be a known issue in Hexo that raw HTLM table code always having problem to be rendered properly. In Hexo, If certain content is causing processing issues, wrap it with the `raw` tag to avoid rendering errors. Therefore, To fix the problem, just add a pair of `raw` tags to wrap the DataFrame section.
```markdwon
{% raw %}
<div>
    <style scoped>
    ...
    </style>
    <table border="1" class="dataframe">
    ...
    </table>
</div>
{% endraw %}
```
### Pretty Tables
Next, add some CSS code to format the DataFrame tables appropriately. I'm using [Next Theme](https://github.com/theme-next/hexo-theme-next) of Hexo which provides a global option for customized CSS code.
Add the customized CSS code in `themes\next\source\css\_costom\custome.styl`. This change will take effect for all DataFrame tables in all posts.
```CSS
// beauty DataFrame 
table.dataframe {
    width: 100%;
    height: 240px;
    display: block;
    overflow: auto;
    font-family: Arial, sans-serif;
    font-size: 13px;
    line-height: 20px;
    text-align: center;
}
table.dataframe th {
  font-weight: bold;
  padding: 4px;
  border-bottom: 0px;
}
table.dataframe td {
  padding: 4px;
  border-bottom: 0px;
}
table.dataframe tr:hover {
  background: #b8d1f3; 
}
```
Done. Here is a sample DataFrame table:

{% raw %}
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total</th>
      <th>Percent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>PuaMode</th>
      <td>8919174</td>
      <td>99.974119</td>
    </tr>
    <tr>
      <th>Census_ProcessorClass</th>
      <td>8884852</td>
      <td>99.589407</td>
    </tr>
    <tr>
      <th>DefaultBrowsersIdentifier</th>
      <td>8488045</td>
      <td>95.141637</td>
    </tr>
    <tr>
      <th>Census_IsFlightingInternal</th>
      <td>7408759</td>
      <td>83.044030</td>
    </tr>
    <tr>
      <th>Census_InternalBatteryType</th>
      <td>6338429</td>
      <td>71.046809</td>
    </tr>
    <tr>
      <th>Census_ThresholdOptIn</th>
      <td>5667325</td>
      <td>63.524472</td>
    </tr>
    <tr>
      <th>Census_IsWIMBootEnabled</th>
      <td>5659703</td>
      <td>63.439038</td>
    </tr>
    <tr>
      <th>SmartScreen</th>
      <td>3177011</td>
      <td>35.610795</td>
    </tr>
    <tr>
      <th>OrganizationIdentifier</th>
      <td>2751518</td>
      <td>30.841487</td>
    </tr>
    <tr>
      <th>SMode</th>
      <td>537759</td>
      <td>6.027686</td>
    </tr>
    <tr>
      <th>CityIdentifier</th>
      <td>325409</td>
      <td>3.647477</td>
    </tr>
    <tr>
      <th>Wdft_IsGamer</th>
      <td>303451</td>
      <td>3.401352</td>
    </tr>
    <tr>
      <th>Wdft_RegionIdentifier</th>
      <td>303451</td>
      <td>3.401352</td>
    </tr>
    <tr>
      <th>Census_InternalBatteryNumberOfCharges</th>
      <td>268755</td>
      <td>3.012448</td>
    </tr>
    <tr>
      <th>Census_FirmwareManufacturerIdentifier</th>
      <td>183257</td>
      <td>2.054109</td>
    </tr>
    <tr>
      <th>Census_IsFlightsDisabled</th>
      <td>160523</td>
      <td>1.799286</td>
    </tr>
    <tr>
      <th>Census_FirmwareVersionIdentifier</th>
      <td>160133</td>
      <td>1.794915</td>
    </tr>
    <tr>
      <th>Census_OEMModelIdentifier</th>
      <td>102233</td>
      <td>1.145919</td>
    </tr>
    <tr>
      <th>Census_OEMNameIdentifier</th>
      <td>95478</td>
      <td>1.070203</td>
    </tr>
    <tr>
      <th>Firewall</th>
      <td>91350</td>
      <td>1.023933</td>
    </tr>
    <tr>
      <th>Census_TotalPhysicalRAM</th>
      <td>80533</td>
      <td>0.902686</td>
    </tr>
    <tr>
      <th>Census_IsAlwaysOnAlwaysConnectedCapable</th>
      <td>71343</td>
      <td>0.799676</td>
    </tr>
    <tr>
      <th>Census_OSInstallLanguageIdentifier</th>
      <td>60084</td>
      <td>0.673475</td>
    </tr>
    <tr>
      <th>IeVerIdentifier</th>
      <td>58894</td>
      <td>0.660137</td>
    </tr>
    <tr>
      <th>Census_PrimaryDiskTotalCapacity</th>
      <td>53016</td>
      <td>0.594251</td>
    </tr>
    <tr>
      <th>Census_SystemVolumeTotalCapacity</th>
      <td>53002</td>
      <td>0.594094</td>
    </tr>
    <tr>
      <th>Census_InternalPrimaryDiagonalDisplaySizeInInches</th>
      <td>47134</td>
      <td>0.528320</td>
    </tr>
    <tr>
      <th>Census_InternalPrimaryDisplayResolutionHorizontal</th>
      <td>46986</td>
      <td>0.526661</td>
    </tr>
    <tr>
      <th>Census_InternalPrimaryDisplayResolutionVertical</th>
      <td>46986</td>
      <td>0.526661</td>
    </tr>
    <tr>
      <th>Census_ProcessorModelIdentifier</th>
      <td>41343</td>
      <td>0.463410</td>
    </tr>
    <tr>
      <th>Census_ProcessorManufacturerIdentifier</th>
      <td>41313</td>
      <td>0.463073</td>
    </tr>
    <tr>
      <th>Census_ProcessorCoreCount</th>
      <td>41306</td>
      <td>0.462995</td>
    </tr>
    <tr>
      <th>AVProductsEnabled</th>
      <td>36221</td>
      <td>0.405998</td>
    </tr>
    <tr>
      <th>AVProductsInstalled</th>
      <td>36221</td>
      <td>0.405998</td>
    </tr>
    <tr>
      <th>AVProductStatesIdentifier</th>
      <td>36221</td>
      <td>0.405998</td>
    </tr>
    <tr>
      <th>IsProtected</th>
      <td>36044</td>
      <td>0.404014</td>
    </tr>
    <tr>
      <th>RtpStateBitfield</th>
      <td>32318</td>
      <td>0.362249</td>
    </tr>
    <tr>
      <th>Census_IsVirtualDevice</th>
      <td>15953</td>
      <td>0.178816</td>
    </tr>
    <tr>
      <th>Census_PrimaryDiskTypeName</th>
      <td>12844</td>
      <td>0.143967</td>
    </tr>
    <tr>
      <th>UacLuaenable</th>
      <td>10838</td>
      <td>0.121482</td>
    </tr>
    <tr>
      <th>Census_ChassisTypeName</th>
      <td>623</td>
      <td>0.006983</td>
    </tr>
    <tr>
      <th>GeoNameIdentifier</th>
      <td>213</td>
      <td>0.002387</td>
    </tr>
    <tr>
      <th>Census_PowerPlatformRoleName</th>
      <td>55</td>
      <td>0.000616</td>
    </tr>
    <tr>
      <th>OsBuildLab</th>
      <td>21</td>
      <td>0.000235</td>
    </tr>
    <tr>
      <th>LocaleEnglishNameIdentifier</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>AvSigVersion</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>OsPlatformSubRelease</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Processor</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>OsVer</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>AppVersion</th>
      <td>0</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>
{% endraw %}
Looks much better.

Reference:
* [Solution about how to post my R code and result on Hexo blog](http://blog.baoduge.com/rproject_hexo/)
* [Happy blogging with Jupyter Notebook](https://blog.baoyukun.win/%E6%8A%80%E6%9C%AF/%E5%89%8D%E7%AB%AF/hbwjnotebook/)
* [Post Jupyter notebooks to your GitHub blog](https://blomadam.github.io/tutorials/2017/04/09/ipynb-to-Jekyll-Post-tools.html)
