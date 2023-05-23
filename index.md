# Time series plots - Obs Vs Mod Data with stats

**Key steps**  
1. Import observed data from a .mat file.  
2. Plot timeseries observed data for three selected sites.  
3. Extract modelled data from tuflow output for the same three sites.  
4. Plot timeseries modelled data for the same three sites.  
5. Plot both obs and mod data in the same plot.  
6. Calculate stats separatrely - Four tests will be used; *Bias, RMSE, MAE, and IOA*.  
7. Add the stats to the timeseries plot.

### Import Libraries


```python
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import pandas as pd
import numpy as np
from tfv.extractor import FvExtractor
import tfv.xarray
import xarray as xr
import datetime as dt
from matplotlib.dates import DateFormatter
from scipy.io import loadmat
import matplotlib.dates as mdates
# define function to convert MATLAB datenum to pandas datetime
def convtime(matlab_datenum):
    return pd.to_datetime(matlab_datenum - 719529, unit='D')
```

## Observed Data


```python
# load .mat file
TEMP_data = r'M:\UWA\A11348_Cockburn_Sound_BCG_Model\2Exectn\2Modelling\1Processing\2Postprocessed\benchmarking\Matlab_scripts\matfiles\PDSP2_data_20230129.mat'
data = loadmat(TEMP_data, simplify_cells=True)
dat = data['PDSP2_data']
```

Let's create data frames for each variable based on selected sites.
Below code chunk creates three dataframes for TEMP,TEMP and SALT. 


```python
# create dictionary of dataframes for each variable
dfs = {}
#sites = ['NORTH', 'CENTRAL', 'SOUTH']
sites = ['NORTH', 'CENTRAL', 'SOUTH','S2', 'S3', 'R2', 'A4', 'A7', 'A10', 'A13','B4', 'B6', 'B8', 'B10', 'C4', 'C6', 'C8']
for var in ['DO', 'TEMP', 'SALT']:
    dfs[var] = pd.DataFrame()
    for site in sites:
        if var in dat[site]:
            if 'Surface' in dat[site][var]:
                df_site = pd.DataFrame({
                    'date': convtime(dat[site][var]['Surface']['date']).round('1S'),
                    'site': site,
                    'depth': 'Surface',
                    var: dat[site][var]['Surface']['data']
                })
                dfs[var] = pd.concat([dfs[var], df_site], ignore_index=True)
            if 'Bottom' in dat[site][var]:
                df_site = pd.DataFrame({
                    'date': convtime(dat[site][var]['Bottom']['date']).round('1S'),
                    'site': site,
                    'depth': 'Bottom',
                    var: dat[site][var]['Bottom']['data']
                })
                dfs[var] = pd.concat([dfs[var], df_site], ignore_index=True)

```

Check the head rows.
It's a good habit to check your data structure before plotting them.



```python
# head of TEMP dataframe
print(dfs['TEMP'].head())
```

                     date   site    depth       TEMP
    0 2013-01-14 12:42:53  NORTH  Surface  24.241109
    1 2013-01-29 11:12:49  NORTH  Surface  23.843754
    2 2013-02-13 09:39:28  NORTH  Surface  24.667787
    3 2013-03-06 10:16:23  NORTH  Surface  24.096557
    4 2013-03-15 08:46:30  NORTH  Surface  22.589213
    


```python
# save dataframes for vars
df_DO = dfs['DO']
df_TEMP = dfs['TEMP']
df_SALT = dfs['SALT']
```

Specify start and end times.  
Feel free to match them with modlled data time periods


```python
# set start and end times
start_time = '2013-01-01'
end_time = '2013-05-30'
```

### Plot Observed Data

Plot observed data for sites = ['NORTH', 'CENTRAL', 'SOUTH']

Variable is Temperature


```python
# filter dataframes by start and end times
df_TEMP_filtered = df_TEMP.loc[(df_TEMP['date'] >= start_time) & (df_TEMP['date'] <= end_time)]

# create subplots for each site
sites = ['NORTH', 'CENTRAL', 'SOUTH']
fig, axs = plt.subplots(nrows=3, ncols=1, figsize=(8, 12))

# plot surface and bottom TEMP data as a scatter plot
for i, site in enumerate(sites):
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    axs[i].scatter(df_site_surface['date'], df_site_surface['TEMP'], s=5, color='#005581', label='Surface Obs')
    axs[i].scatter(df_site_bottom['date'], df_site_bottom['TEMP'], s=5, color='#D4D00F', label='Bottom Obs')
    axs[i].set_title(site)
    axs[i].set_ylim([14,28])
    axs[i].set_yticks([16,18,20,22,24,26,28])
    axs[i].set_ylabel('TEMP (°C)')
    axs[i].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
    axs[i].tick_params(axis='x', rotation=60)
    axs[i].legend().remove()

# add common legend under the last subplot
handles, labels = axs[0].get_legend_handles_labels()
fig.legend(handles, labels, loc='lower center', ncol=2)

fig.subplots_adjust(hspace=0.5)
#plt.show()
#plt.savefig('figures/TEMP/temp_sites1_time_series_obs_TEMP.png')

```


    
![png](Tuflowfv_validation_statsV2_files/Tuflowfv_validation_statsV2_15_0.png)
    


## Modelled data


```python
ds2 = xr.open_dataset(r"K:\scratchers\A11348_Cockburn_Sound_BCG_Model\output\CSound_WQM\WQ_DO_002\CSound_20130101_20130601_WQ_DO_002.nc", decode_times=False)
ds2
ds2.tfv
```


<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body[data-theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:      (Time: 2857, NumLayerFaces3D: 534080, NumCells2D: 30211,
                  NumCells3D: 503869)
Coordinates:
  * Time         (Time) datetime64[ns] 2013-01-02 ... 2013-05-01
Dimensions without coordinates: NumLayerFaces3D, NumCells2D, NumCells3D
Data variables:
    ResTime      (Time) float64 2.016e+05 2.016e+05 ... 2.045e+05 2.045e+05
    layerface_Z  (Time, NumLayerFaces3D) float32 ...
    stat         (Time, NumCells2D) int32 ...
    H            (Time, NumCells2D) float32 ...
    V_x          (Time, NumCells3D) float32 ...
    V_y          (Time, NumCells3D) float32 ...
    SAL          (Time, NumCells3D) float32 ...
    TEMP         (Time, NumCells3D) float32 ...
    TRACE_1      (Time, NumCells3D) float32 ...
Attributes:
    Origin:     Created by TUFLOWFV
    Type:       Cell-centred TUFLOWFV output
    spherical:  true
    Dry depth:  0.01</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-ad63dae0-dc55-4d83-bc1e-66a9e6eae2c6' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-ad63dae0-dc55-4d83-bc1e-66a9e6eae2c6' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>Time</span>: 2857</li><li><span>NumLayerFaces3D</span>: 534080</li><li><span>NumCells2D</span>: 30211</li><li><span>NumCells3D</span>: 503869</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-61ea4823-8e27-44ad-ba5f-4ac8284cc988' class='xr-section-summary-in' type='checkbox'  checked><label for='section-61ea4823-8e27-44ad-ba5f-4ac8284cc988' class='xr-section-summary' >Coordinates: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>Time</span></div><div class='xr-var-dims'>(Time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2013-01-02 ... 2013-05-01</div><input id='attrs-f6a19c10-c723-4a36-b6e3-41d5ab73e3d3' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-f6a19c10-c723-4a36-b6e3-41d5ab73e3d3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6a9a7be9-e328-4b49-b08c-0adca326f2f8' class='xr-var-data-in' type='checkbox'><label for='data-6a9a7be9-e328-4b49-b08c-0adca326f2f8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2013-01-02T00:00:00.000000000&#x27;, &#x27;2013-01-02T01:00:00.000000000&#x27;,
       &#x27;2013-01-02T02:00:00.000000000&#x27;, ..., &#x27;2013-04-30T22:00:00.000000000&#x27;,
       &#x27;2013-04-30T23:00:00.000000000&#x27;, &#x27;2013-05-01T00:00:00.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-b003a378-fb57-4aac-baaa-e07254df4c8c' class='xr-section-summary-in' type='checkbox'  checked><label for='section-b003a378-fb57-4aac-baaa-e07254df4c8c' class='xr-section-summary' >Data variables: <span>(9)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>ResTime</span></div><div class='xr-var-dims'>(Time)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>2.016e+05 2.016e+05 ... 2.045e+05</div><input id='attrs-6368be25-fb32-4de6-9670-2a82afe2bd31' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-6368be25-fb32-4de6-9670-2a82afe2bd31' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-02eb7c6c-75eb-49f9-8973-095a50e69721' class='xr-var-data-in' type='checkbox'><label for='data-02eb7c6c-75eb-49f9-8973-095a50e69721' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>output time relative to 01/01/1990 00:00:00</dd><dt><span>units :</span></dt><dd>hours since 1990-01-01T00:00:00</dd></dl></div><div class='xr-var-data'><pre>array([201648., 201649., 201650., ..., 204502., 204503., 204504.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>layerface_Z</span></div><div class='xr-var-dims'>(Time, NumLayerFaces3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-2be87297-5973-431f-a78a-a228b2799b53' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-2be87297-5973-431f-a78a-a228b2799b53' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b3c70167-8d37-4ca7-a048-b04c00131237' class='xr-var-data-in' type='checkbox'><label for='data-b3c70167-8d37-4ca7-a048-b04c00131237' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Layer Face Z-Coordinates</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>[1525866560 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>stat</span></div><div class='xr-var-dims'>(Time, NumCells2D)</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-41be0deb-7f60-4e08-8892-78dc231b9e53' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-41be0deb-7f60-4e08-8892-78dc231b9e53' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-87187648-5541-4f19-b02a-2d9913979f6a' class='xr-var-data-in' type='checkbox'><label for='data-87187648-5541-4f19-b02a-2d9913979f6a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Cell wet/dry status</dd><dt><span>units :</span></dt><dd>boolean</dd></dl></div><div class='xr-var-data'><pre>[86312827 values with dtype=int32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>H</span></div><div class='xr-var-dims'>(Time, NumCells2D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-cbb03196-2a5a-43e1-a178-3f97a4b91e0e' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-cbb03196-2a5a-43e1-a178-3f97a4b91e0e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-039fb3eb-ddc0-462a-b2b3-f1818ea27f7c' class='xr-var-data-in' type='checkbox'><label for='data-039fb3eb-ddc0-462a-b2b3-f1818ea27f7c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>water surface elevation</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>[86312827 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>V_x</span></div><div class='xr-var-dims'>(Time, NumCells3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-4dce77d9-a5f5-47a4-a45e-74a10604e7a6' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-4dce77d9-a5f5-47a4-a45e-74a10604e7a6' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a96062ea-d32c-4316-be8d-2fd4d79049df' class='xr-var-data-in' type='checkbox'><label for='data-a96062ea-d32c-4316-be8d-2fd4d79049df' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>x_velocity</dd><dt><span>units :</span></dt><dd>m s^-1</dd></dl></div><div class='xr-var-data'><pre>[1439553733 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>V_y</span></div><div class='xr-var-dims'>(Time, NumCells3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-afb8500c-a5c8-486c-b000-0328db6fc49b' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-afb8500c-a5c8-486c-b000-0328db6fc49b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-199baa70-b227-4f34-bbda-9fb32fc3bcd2' class='xr-var-data-in' type='checkbox'><label for='data-199baa70-b227-4f34-bbda-9fb32fc3bcd2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>y_velocity</dd><dt><span>units :</span></dt><dd>m s^-1</dd></dl></div><div class='xr-var-data'><pre>[1439553733 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>SAL</span></div><div class='xr-var-dims'>(Time, NumCells3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-d15ccdc7-f6c5-4b6c-b6d4-7266562f3b67' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-d15ccdc7-f6c5-4b6c-b6d4-7266562f3b67' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f9f3e069-d75e-49ed-a10d-dc309bf1a27f' class='xr-var-data-in' type='checkbox'><label for='data-f9f3e069-d75e-49ed-a10d-dc309bf1a27f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>salinity</dd><dt><span>units :</span></dt><dd>psu</dd></dl></div><div class='xr-var-data'><pre>[1439553733 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>TEMP</span></div><div class='xr-var-dims'>(Time, NumCells3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-444d7c73-89c6-4d47-8d18-b3b2380b46da' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-444d7c73-89c6-4d47-8d18-b3b2380b46da' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4ddf1cd7-fc7e-4160-b9b3-c2f4a86a96ee' class='xr-var-data-in' type='checkbox'><label for='data-4ddf1cd7-fc7e-4160-b9b3-c2f4a86a96ee' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>temperature</dd><dt><span>units :</span></dt><dd>degrees celsius</dd></dl></div><div class='xr-var-data'><pre>[1439553733 values with dtype=float32]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>TRACE_1</span></div><div class='xr-var-dims'>(Time, NumCells3D)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-4709cf23-d1f9-4f6c-ae2d-fdf34191ba73' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-4709cf23-d1f9-4f6c-ae2d-fdf34191ba73' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-5cffd2de-c17f-4e74-9314-96ac56d7783e' class='xr-var-data-in' type='checkbox'><label for='data-5cffd2de-c17f-4e74-9314-96ac56d7783e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>tracer 1 concentration</dd><dt><span>units :</span></dt><dd>units m^-3</dd></dl></div><div class='xr-var-data'><pre>[1439553733 values with dtype=float32]</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-a7a97b73-2610-460c-abfc-c251e9eaeb51' class='xr-section-summary-in' type='checkbox'  ><label for='section-a7a97b73-2610-460c-abfc-c251e9eaeb51' class='xr-section-summary' >Indexes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>Time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-ead474d0-1486-40b0-8de3-b892167a868f' class='xr-index-data-in' type='checkbox'/><label for='index-ead474d0-1486-40b0-8de3-b892167a868f' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;2013-01-02 00:00:00&#x27;, &#x27;2013-01-02 01:00:00&#x27;,
               &#x27;2013-01-02 02:00:00&#x27;, &#x27;2013-01-02 03:00:00&#x27;,
               &#x27;2013-01-02 04:00:00&#x27;, &#x27;2013-01-02 05:00:00&#x27;,
               &#x27;2013-01-02 06:00:00&#x27;, &#x27;2013-01-02 07:00:00&#x27;,
               &#x27;2013-01-02 08:00:00&#x27;, &#x27;2013-01-02 09:00:00&#x27;,
               ...
               &#x27;2013-04-30 15:00:00&#x27;, &#x27;2013-04-30 16:00:00&#x27;,
               &#x27;2013-04-30 17:00:00&#x27;, &#x27;2013-04-30 18:00:00&#x27;,
               &#x27;2013-04-30 19:00:00&#x27;, &#x27;2013-04-30 20:00:00&#x27;,
               &#x27;2013-04-30 21:00:00&#x27;, &#x27;2013-04-30 22:00:00&#x27;,
               &#x27;2013-04-30 23:00:00&#x27;, &#x27;2013-05-01 00:00:00&#x27;],
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;Time&#x27;, length=2857, freq=None))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-3f7c9f44-1b6f-4a3d-bc05-4345384b5a44' class='xr-section-summary-in' type='checkbox'  checked><label for='section-3f7c9f44-1b6f-4a3d-bc05-4345384b5a44' class='xr-section-summary' >Attributes: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>Origin :</span></dt><dd>Created by TUFLOWFV</dd><dt><span>Type :</span></dt><dd>Cell-centred TUFLOWFV output</dd><dt><span>spherical :</span></dt><dd>true</dd><dt><span>Dry depth :</span></dt><dd>0.01</dd></dl></div></li></ul></div></div>





    <tfv.xarray.TfvAccessor at 0x2777b16bcd0>




```python
# read location data from csv file
loc_data = r'M:\UWA\A11348_Cockburn_Sound_BCG_Model\2Exectn\2Modelling\1Processing\2Postprocessed\calibration_wq\data\PDSP2_sites.csv'
df = pd.read_csv(loc_data, delimiter=',', usecols=['X', 'Y', 'Site'])
locs = {}
for i, row in df.iterrows():
    locs[row['Site']] = (row['X'], row['Y'])

# set time range
start_time = dt.datetime(2013, 1, 2, 0, 0, 0)  # replace with your start time
end_time = dt.datetime(2013, 5, 30, 0, 0, 0)  # replace with your end time
```


```python
ts2_top = ds2.tfv.get_timeseries(['TEMP'], locs, datum='depth', limits=(0, 1))
ts2_btm = ds2.tfv.get_timeseries(['TEMP'], locs, datum='height', limits=(0, 1))
```

    Extracting timeseries, please wait: 100%|[34m██████████████████████████████████████████[0m| 2857/2857 [00:59<00:00, 48.31it/s][0m
    Extracting timeseries, please wait: 100%|[34m██████████████████████████████████████████[0m| 2857/2857 [00:59<00:00, 47.68it/s][0m
    


```python
for i, site in enumerate(['NORTH', 'CENTRAL', 'SOUTH']):
    # get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
```


```python
#Check the array structure
btmts2_data
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body[data-theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;TEMP&#x27; (variable: 1, Time: 2857)&gt;
array([[24.70000076, 24.69987583, 24.69982147, ..., 21.79384327,
        21.79740143, 21.80286503]])
Coordinates:
  * Time      (Time) datetime64[ns] 2013-01-02 ... 2013-05-01
    Location  &lt;U23 &#x27;SOUTH&#x27;
    x         float64 115.7
    y         float64 -32.25
    z         float32 -19.58
  * variable  (variable) object &#x27;TEMP&#x27;
Attributes:
    Origin:     Timeseries extracted from TUFLOWFV cell-centered output using...
    Type:       Timeseries cell from TUFLOWFV Output
    spherical:  true
    Dry depth:  0.01
    Datum:      height
    Limits:     (0, 1)
    Agg Fn:     mean</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'TEMP'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>variable</span>: 1</li><li><span class='xr-has-index'>Time</span>: 2857</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-33f5c132-0a00-4e3c-8d7b-d6826bf6ca7f' class='xr-array-in' type='checkbox' checked><label for='section-33f5c132-0a00-4e3c-8d7b-d6826bf6ca7f' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>24.7 24.7 24.7 24.7 24.7 24.68 ... 21.79 21.79 21.79 21.79 21.8 21.8</span></div><div class='xr-array-data'><pre>array([[24.70000076, 24.69987583, 24.69982147, ..., 21.79384327,
        21.79740143, 21.80286503]])</pre></div></div></li><li class='xr-section-item'><input id='section-219dd687-f5ac-4c8d-b4f3-3ba58190f441' class='xr-section-summary-in' type='checkbox'  checked><label for='section-219dd687-f5ac-4c8d-b4f3-3ba58190f441' class='xr-section-summary' >Coordinates: <span>(6)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>Time</span></div><div class='xr-var-dims'>(Time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2013-01-02 ... 2013-05-01</div><input id='attrs-dcfdc532-417f-4fd9-84df-c97dec874d97' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-dcfdc532-417f-4fd9-84df-c97dec874d97' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-57f04bb5-1267-4c87-9171-11401fc6ef35' class='xr-var-data-in' type='checkbox'><label for='data-57f04bb5-1267-4c87-9171-11401fc6ef35' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2013-01-02T00:00:00.000000000&#x27;, &#x27;2013-01-02T01:00:00.000000000&#x27;,
       &#x27;2013-01-02T02:00:00.000000000&#x27;, ..., &#x27;2013-04-30T22:00:00.000000000&#x27;,
       &#x27;2013-04-30T23:00:00.000000000&#x27;, &#x27;2013-05-01T00:00:00.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>Location</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>&lt;U23</div><div class='xr-var-preview xr-preview'>&#x27;SOUTH&#x27;</div><input id='attrs-d630958d-ea86-4ed0-bb9d-ae70ed7a3ab8' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-d630958d-ea86-4ed0-bb9d-ae70ed7a3ab8' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-30645e51-5b2d-4416-b998-c06c78d9095d' class='xr-var-data-in' type='checkbox'><label for='data-30645e51-5b2d-4416-b998-c06c78d9095d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(&#x27;SOUTH&#x27;, dtype=&#x27;&lt;U23&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>x</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>115.7</div><input id='attrs-01b57d9a-33f6-400e-b045-29eb92e73fde' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-01b57d9a-33f6-400e-b045-29eb92e73fde' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f9063eb6-e501-453d-a857-1f125a86026e' class='xr-var-data-in' type='checkbox'><label for='data-f9063eb6-e501-453d-a857-1f125a86026e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(115.726219)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>y</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-32.25</div><input id='attrs-14f14162-35d5-47d2-a6f4-9a247aea26fb' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-14f14162-35d5-47d2-a6f4-9a247aea26fb' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4a02e8f8-99a6-4035-8e55-4b32c275c2e5' class='xr-var-data-in' type='checkbox'><label for='data-4a02e8f8-99a6-4035-8e55-4b32c275c2e5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(-32.252956)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>z</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>-19.58</div><input id='attrs-a4e774fd-9da0-4e31-ae73-9126d42b170f' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a4e774fd-9da0-4e31-ae73-9126d42b170f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-016c2f81-83f2-41f0-9f7d-497664308059' class='xr-var-data-in' type='checkbox'><label for='data-016c2f81-83f2-41f0-9f7d-497664308059' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(-19.576088, dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>variable</span></div><div class='xr-var-dims'>(variable)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>&#x27;TEMP&#x27;</div><input id='attrs-20c09e1f-b548-49f2-98fd-e288a3eb8055' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-20c09e1f-b548-49f2-98fd-e288a3eb8055' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-fcf0c41d-587c-468f-a05b-868601609606' class='xr-var-data-in' type='checkbox'><label for='data-fcf0c41d-587c-468f-a05b-868601609606' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;TEMP&#x27;], dtype=object)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-4a046c1c-d75d-4855-800f-64eddf5a7cd6' class='xr-section-summary-in' type='checkbox'  ><label for='section-4a046c1c-d75d-4855-800f-64eddf5a7cd6' class='xr-section-summary' >Indexes: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>Time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-b3267296-d38b-475c-8956-6aff9f353eee' class='xr-index-data-in' type='checkbox'/><label for='index-b3267296-d38b-475c-8956-6aff9f353eee' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;2013-01-02 00:00:00&#x27;, &#x27;2013-01-02 01:00:00&#x27;,
               &#x27;2013-01-02 02:00:00&#x27;, &#x27;2013-01-02 03:00:00&#x27;,
               &#x27;2013-01-02 04:00:00&#x27;, &#x27;2013-01-02 05:00:00&#x27;,
               &#x27;2013-01-02 06:00:00&#x27;, &#x27;2013-01-02 07:00:00&#x27;,
               &#x27;2013-01-02 08:00:00&#x27;, &#x27;2013-01-02 09:00:00&#x27;,
               ...
               &#x27;2013-04-30 15:00:00&#x27;, &#x27;2013-04-30 16:00:00&#x27;,
               &#x27;2013-04-30 17:00:00&#x27;, &#x27;2013-04-30 18:00:00&#x27;,
               &#x27;2013-04-30 19:00:00&#x27;, &#x27;2013-04-30 20:00:00&#x27;,
               &#x27;2013-04-30 21:00:00&#x27;, &#x27;2013-04-30 22:00:00&#x27;,
               &#x27;2013-04-30 23:00:00&#x27;, &#x27;2013-05-01 00:00:00&#x27;],
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;Time&#x27;, length=2857, freq=None))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>variable</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-75bdfd71-912d-4932-8323-17da2ba38f72' class='xr-index-data-in' type='checkbox'/><label for='index-75bdfd71-912d-4932-8323-17da2ba38f72' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([&#x27;TEMP&#x27;], dtype=&#x27;object&#x27;, name=&#x27;variable&#x27;))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-53522240-7e8a-44f3-b47a-afb63047988a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-53522240-7e8a-44f3-b47a-afb63047988a' class='xr-section-summary' >Attributes: <span>(7)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>Origin :</span></dt><dd>Timeseries extracted from TUFLOWFV cell-centered output using `tfv` python tools</dd><dt><span>Type :</span></dt><dd>Timeseries cell from TUFLOWFV Output</dd><dt><span>spherical :</span></dt><dd>true</dd><dt><span>Dry depth :</span></dt><dd>0.01</dd><dt><span>Datum :</span></dt><dd>height</dd><dt><span>Limits :</span></dt><dd>(0, 1)</dd><dt><span>Agg Fn :</span></dt><dd>mean</dd></dl></div></li></ul></div></div>



### Plot modelled data


```python
fig, axs = plt.subplots(nrows=3, ncols=1, figsize=(8, 12))
for i, site in enumerate(['NORTH', 'CENTRAL', 'SOUTH']):
    # get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    axs[i].plot_date(topts2_data.Time.values, topts2_data.values[0], fmt='-', color='#1ABDC9',label='Surface Model_TS2',linewidth=0.7)
    axs[i].plot_date(btmts2_data.Time.values, btmts2_data.values[0], fmt='--', color='#1ABDC9',label='Bottom Model_TS2',linewidth=0.7)
    # set y-axis limits and ticks, and add y-axis label
    axs[i].set_ylim([14,28])
    axs[i].set_yticks([16,18,20,22,24,26,28])
    axs[i].set_ylabel('TEMP (°C)')
    
    # set subplot title and x-axis label to date format
    axs[i].set_title(site)
    axs[i].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
    axs[i].tick_params(axis='x', rotation=45)
    
# add common legend under the last subplot
handles, labels = axs[2].get_legend_handles_labels()
fig.legend(handles, labels, loc='lower center', ncol=3)

# adjust subplot spacing and save the figure
fig.subplots_adjust(hspace=0.5)
```


    
![png](Tuflowfv_validation_statsV2_files/Tuflowfv_validation_statsV2_23_0.png)
    


## Plot Mod and Obs data in one plot


```python
# loop through the site lists and create the subplots
fig, axs = plt.subplots(nrows=3, ncols=1, figsize=(8, 12))

# plot observed data
for i, site in enumerate(['NORTH', 'CENTRAL', 'SOUTH']):
    # filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # plot observed data as scatter plot
    axs[i].scatter(df_site_surface['date'], df_site_surface['TEMP'], s=5, color='#005581', label='Surface Obs')
    axs[i].scatter(df_site_bottom['date'], df_site_bottom['TEMP'], s=5, color='#D4D00F', label='Bottom Obs')

    # get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP') 
   
       
  # plot modelled data
    axs[i].plot_date(topts2_data.Time.values, topts2_data.values[0], fmt='-', color='#1ABDC9',label='Surface Model_TS2',linewidth=0.7)
    axs[i].plot_date(btmts2_data.Time.values, btmts2_data.values[0], fmt='--', color='#1ABDC9',label='Bottom Model_TS2',linewidth=0.7)
       
    # set y-axis limits and ticks, and add y-axis label
    axs[i].set_ylim([14,28])
    axs[i].set_yticks([16,18,20,22,24,26,28])
    axs[i].set_ylabel('TEMP (°C)')
    
    # set subplot title and x-axis label to date format
    axs[i].set_title(site)
    axs[i].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
    axs[i].tick_params(axis='x', rotation=45)
    
# add common legend under the last subplot
handles, labels = axs[2].get_legend_handles_labels()
fig.legend(handles, labels, loc='lower center', ncol=4)

# adjust subplot spacing and save the figure
fig.subplots_adjust(hspace=0.6)
#plt.savefig('figures/TEMP/temp_sites1_time_series_obs_vs_mod_TS11.jpeg')


```


    
![png](Tuflowfv_validation_statsV2_files/Tuflowfv_validation_statsV2_25_0.png)
    


# Calculate stats

## BIAS  

The bias measures the average difference between the modeled and observed values. It provides an indication of the systematic over- or underestimation of the model compared to the observations. The bias is calculated as the mean of the differences between the modeled and observed values.

**Bias = mean(modeled values - observed values)**

Step 1; Calculate mean values for observed data 


```python
surface_mean_per_site = {}
bottom_mean_per_site = {}

# Calculate mean values for surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # Calculate mean values for surface and bottom data
    surface_mean = df_site_surface['TEMP'].mean()
    bottom_mean = df_site_bottom['TEMP'].mean()
    
    # Store mean values in dictionaries
    surface_mean_per_site[site] = surface_mean
    bottom_mean_per_site[site] = bottom_mean
    
    # Print mean values for observed data
    print(f"Observed Mean at {site} (Surface): {surface_mean:.2f}")
    print(f"Observed Mean at {site} (Bottom): {bottom_mean:.2f}")

```

    Observed Mean at NORTH (Surface): 22.79
    Observed Mean at NORTH (Bottom): 22.58
    Observed Mean at CENTRAL (Surface): 22.51
    Observed Mean at CENTRAL (Bottom): 22.16
    Observed Mean at SOUTH (Surface): 22.45
    Observed Mean at SOUTH (Bottom): 22.01
    

Step 2; Calculate mean values for modelled data 


```python
surface_model_mean_per_site = {}
bottom_model_mean_per_site = {}

# Calculate mean values for modelled surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')

    # Calculate mean values for modelled surface and bottom data
    surface_model_mean = topts2_data.mean().item()
    bottom_model_mean = btmts2_data.mean().item()

    # Store mean values in dictionaries
    surface_model_mean_per_site[site] = surface_model_mean
    bottom_model_mean_per_site[site] = bottom_model_mean

    # Print mean values for modelled data
    print(f"Modelled Mean at {site} (Surface): {surface_model_mean:.2f}")
    print(f"Modelled Mean at {site} (Bottom): {bottom_model_mean:.2f}")

```

    Modelled Mean at NORTH (Surface): 23.60
    Modelled Mean at NORTH (Bottom): 23.37
    Modelled Mean at CENTRAL (Surface): 23.64
    Modelled Mean at CENTRAL (Bottom): 23.38
    Modelled Mean at SOUTH (Surface): 23.75
    Modelled Mean at SOUTH (Bottom): 23.27
    

Step 3; Calculate BIAS


```python
surface_bias_per_site = {}
bottom_bias_per_site = {}

# Calculate bias for surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Calculate bias for surface data
    surface_bias = surface_model_mean_per_site[site] - surface_mean_per_site[site]
    surface_bias_per_site[site] = surface_bias

    # Calculate bias for bottom data
    bottom_bias = bottom_model_mean_per_site[site] - bottom_mean_per_site[site]
    bottom_bias_per_site[site] = bottom_bias

    # Print bias values
    print(f"Bias at {site} (Surface): {surface_bias:.2f}")
    print(f"Bias at {site} (Bottom): {bottom_bias:.2f}")

```

    Bias at NORTH (Surface): 0.81
    Bias at NORTH (Bottom): 0.79
    Bias at CENTRAL (Surface): 1.14
    Bias at CENTRAL (Bottom): 1.22
    Bias at SOUTH (Surface): 1.30
    Bias at SOUTH (Bottom): 1.26
    

## Root Mean Square Error (RMSE)   

RMSE measures the average magnitude of the differences between the modeled and observed values. It quantifies the overall error or discrepancy between the model and the observations, considering both the magnitude and direction of the differences. The RMSE is calculated as the square root of the mean squared differences between the modeled and observed values.

**RMSE = sqrt(mean((modeled values - observed values)^2))**

Calculate RMSE


```python
from sklearn.metrics import mean_squared_error

surface_rmse_per_site = {}
bottom_rmse_per_site = {}

# Calculate RMSE for surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # Convert mean values to numpy arrays
    surface_mean = np.array(surface_model_mean_per_site[site])
    bottom_mean = np.array(bottom_model_mean_per_site[site])

    # Calculate RMSE for surface data
    surface_obs = df_site_surface['TEMP'].values.astype(float)
    surface_rmse = np.sqrt(mean_squared_error(surface_obs, np.full(surface_obs.shape, surface_mean)))
    surface_rmse_per_site[site] = surface_rmse

    # Calculate RMSE for bottom data
    bottom_obs = df_site_bottom['TEMP'].values.astype(float)
    bottom_rmse = np.sqrt(mean_squared_error(bottom_obs, np.full(bottom_obs.shape, bottom_mean)))
    bottom_rmse_per_site[site] = bottom_rmse

    # Print RMSE values
    print(f"RMSE at {site} (Surface): {surface_rmse:.2f}")
    print(f"RMSE at {site} (Bottom): {bottom_rmse:.2f}")

```

    RMSE at NORTH (Surface): 1.92
    RMSE at NORTH (Bottom): 1.81
    RMSE at CENTRAL (Surface): 1.68
    RMSE at CENTRAL (Bottom): 1.65
    RMSE at SOUTH (Surface): 1.82
    RMSE at SOUTH (Bottom): 1.66
    

## Mean Absolute Error (MAE)  

MAE measures the average magnitude of the differences between the modeled and observed values, ignoring the direction of the differences. It provides a measure of the average absolute deviation of the model from the observations. The MAE is calculated as the mean of the absolute differences between the modeled and observed values.

**MAE = mean(abs(modeled values - observed values))**



```python
Calculate MAE
```


```python
surface_mae_per_site = {}
bottom_mae_per_site = {}

# Calculate MAE for surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # Convert mean values to numpy arrays
    surface_mean = np.array(surface_mean_per_site[site])
    bottom_mean = np.array(bottom_mean_per_site[site])
    
    # Calculate MAE for surface data
    surface_obs = df_site_surface['TEMP'].values.astype(float)
    surface_mae = mean_absolute_error(surface_obs, np.full(surface_obs.shape, surface_mean))
    surface_mae_per_site[site] = surface_mae

    # Calculate MAE for bottom data
    bottom_obs = df_site_bottom['TEMP'].values.astype(float)
    bottom_mae = mean_absolute_error(bottom_obs, np.full(bottom_obs.shape, bottom_mean))
    bottom_mae_per_site[site] = bottom_mae

    # Print MAE values
    print(f"MAE at {site} (Surface): {surface_mae:.2f}")
    print(f"MAE at {site} (Bottom): {bottom_mae:.2f}")

```

    MAE at NORTH (Surface): 1.42
    MAE at NORTH (Bottom): 1.33
    MAE at CENTRAL (Surface): 0.85
    MAE at CENTRAL (Bottom): 0.78
    MAE at SOUTH (Surface): 0.94
    MAE at SOUTH (Bottom): 0.73
    

## Index of Agreement (IOA)  

IOA is a statistical measure that quantifies the agreement between the modeled and observed values, taking into account both the bias and the variability of the data. It ranges from 0 to 1, with values closer to 1 indicating a better agreement between the model and the observations. IOA is calculated using the formula:

**IOA = 1 - (sum((modeled values - observed values)$^2$) / sum((|modeled values - mean(observed values)| + |observed values - mean(observed values)|)$^2$))**

The numerator represents the squared differences between the modeled and observed values, while the denominator represents the sum of the squared differences between the modeled values and the mean observed value, and the squared differences between the observed values and their mean.


```python
Calculate IOA
```


```python
surface_ioa_per_site = {}
bottom_ioa_per_site = {}

# Calculate IOA for surface and bottom data at each site
for site in ['NORTH', 'CENTRAL', 'SOUTH']:
    # Filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]

    # Convert mean values to numpy arrays
    surface_mean = np.array(surface_model_mean_per_site[site])
    bottom_mean = np.array(bottom_model_mean_per_site[site])

    # Calculate IOA for surface data
    surface_obs = df_site_surface['TEMP'].values.astype(float)
    numerator = np.sum((surface_mean - surface_obs) ** 2)
    denominator = np.sum((np.abs(surface_mean - np.mean(surface_obs)) + np.abs(surface_obs - np.mean(surface_obs))) ** 2)
    surface_ioa = 1 - (numerator / denominator)
    surface_ioa_per_site[site] = surface_ioa

    # Calculate IOA for bottom data
    bottom_obs = df_site_bottom['TEMP'].values.astype(float)
    numerator = np.sum((bottom_mean - bottom_obs) ** 2)
    denominator = np.sum((np.abs(bottom_mean - np.mean(bottom_obs)) + np.abs(bottom_obs - np.mean(bottom_obs))) ** 2)
    bottom_ioa = 1 - (numerator / denominator)
    bottom_ioa_per_site[site] = bottom_ioa

    # Print IOA values
    print(f"IOA at {site} (Surface): {surface_ioa:.2f}")
    print(f"IOA at {site} (Bottom): {bottom_ioa:.2f}")

```

    IOA at NORTH (Surface): 0.38
    IOA at NORTH (Bottom): 0.39
    IOA at CENTRAL (Surface): 0.41
    IOA at CENTRAL (Bottom): 0.41
    IOA at SOUTH (Surface): 0.43
    IOA at SOUTH (Bottom): 0.40
    

### Add BIAS, RMSE, MAE and IOA stats to the figure


```python
# loop through the site lists and create the subplots
fig, axs = plt.subplots(nrows=3, ncols=1, figsize=(8, 12))

# plot observed data
for i, site in enumerate(['NORTH', 'CENTRAL', 'SOUTH']):
    # filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # plot observed data as scatter plot
    axs[i].scatter(df_site_surface['date'], df_site_surface['TEMP'], s=5, color='#005581', label='Surface Obs')
    axs[i].scatter(df_site_bottom['date'], df_site_bottom['TEMP'], s=5, color='#D4D00F', label='Bottom Obs')

    # get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP') 

    # Calculate bias for surface and bottom data
    surface_bias = surface_model_mean_per_site[site] - surface_mean_per_site[site]
    bottom_bias = bottom_model_mean_per_site[site] - bottom_mean_per_site[site]

    # Calculate RMSE for surface and bottom data
    surface_rmse = surface_rmse_per_site[site]
    bottom_rmse = bottom_rmse_per_site[site]

    # Calculate MAE for surface and bottom data
    surface_mae = surface_mae_per_site[site]
    bottom_mae = bottom_mae_per_site[site]
    
    # Calculate IOA for surface and bottom data
    surface_ioa = surface_ioa_per_site[site]
    bottom_ioa = bottom_ioa_per_site[site]

    # plot modelled data
    axs[i].plot_date(topts2_data.Time.values, topts2_data.values[0], fmt='-', color='#1ABDC9',label='Surface Model_TS2',linewidth=0.7)
    axs[i].plot_date(btmts2_data.Time.values, btmts2_data.values[0], fmt='--', color='#1ABDC9',label='Bottom Model_TS2',linewidth=0.7)

    # Add a box with all the labels
    bbox_props = dict(boxstyle='round', facecolor='white', edgecolor='black', linewidth=0.5)
    label_text = f"Bias (Surface): {surface_bias:.2f}\nRMSE (Surface): {surface_rmse:.2f}\nMAE (Surface): {surface_mae:.2f}\nIOA (Surface): {surface_ioa:.2f}\nBias (Bottom): {bottom_bias:.2f}\nRMSE (Bottom): {bottom_rmse:.2f}\nMAE (Bottom): {bottom_mae:.2f}\nIOA (Bottom): {bottom_ioa:.2f}"
    axs[i].text(0.83, 0.6, label_text, transform=axs[i].transAxes, fontsize=7, bbox=bbox_props)
  
    # set y-axis limits and ticks, and add y-axis label
    axs[i].set_ylim([14, 28])
    axs[i].set_yticks([16, 18, 20, 22, 24, 26, 28])
    axs[i].set_ylabel('TEMP (°C)')
    
    # set subplot title and x-axis label to date format
    axs[i].set_title(site)
    axs[i].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
    axs[i].tick_params(axis='x', rotation=45)
    
# add common legend under the last subplot
handles, labels = axs[2].get_legend_handles_labels()
fig.legend(handles, labels, loc='lower center', ncol=4)

# adjust subplot spacing and save the figure
fig.subplots_adjust(hspace=0.6)
#plt.savefig('figures/TEMP/temp_sites1_time_series_obs_vs_mod_TS11.jpeg')

```


    
![png](Tuflowfv_validation_statsV2_files/Tuflowfv_validation_statsV2_44_0.png)
    


## Tidy up the figure    
Move the stat table outside the plot area, add a title to the stat box


```python
# loop through the site lists and create the subplots
fig, axs = plt.subplots(nrows=3, ncols=1, figsize=(12, 12))

# plot observed data
for i, site in enumerate(['NORTH', 'CENTRAL', 'SOUTH']):
    # filter dataframes by site and depth
    df_site_surface = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Surface')]
    df_site_bottom = df_TEMP_filtered.loc[(df_TEMP_filtered['site'] == site) & (df_TEMP_filtered['depth'] == 'Bottom')]
    
    # plot observed data as scatter plot
    axs[i].scatter(df_site_surface['date'], df_site_surface['TEMP'], s=5, color='#005581', label='Surface Obs')
    axs[i].scatter(df_site_bottom['date'], df_site_bottom['TEMP'], s=5, color='#D4D00F', label='Bottom Obs')

    # get the top and bottom data for the current site
    btmts2_data = ts2_btm.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP')
    topts2_data = ts2_top.sel(Location=site).sel(Time=slice(start_time, end_time)).to_array(name='TEMP') 

    # Calculate bias for surface and bottom data
    surface_bias = surface_model_mean_per_site[site] - surface_mean_per_site[site]
    bottom_bias = bottom_model_mean_per_site[site] - bottom_mean_per_site[site]

    # Calculate RMSE for surface and bottom data
    surface_rmse = surface_rmse_per_site[site]
    bottom_rmse = bottom_rmse_per_site[site]

    # Calculate MAE for surface and bottom data
    surface_mae = surface_mae_per_site[site]
    bottom_mae = bottom_mae_per_site[site]
    
    # Calculate IOA for surface and bottom data
    surface_ioa = surface_ioa_per_site[site]
    bottom_ioa = bottom_ioa_per_site[site]

    # plot modelled data
    axs[i].plot_date(topts2_data.Time.values, topts2_data.values[0], fmt='-', color='#1ABDC9',label='Surface Model_TS2',linewidth=0.7)
    axs[i].plot_date(btmts2_data.Time.values, btmts2_data.values[0], fmt='--', color='#1ABDC9',label='Bottom Model_TS2',linewidth=0.7)

    # Add a box with all the labels
    bbox_props = dict(boxstyle='round', facecolor='white', edgecolor='black', linewidth=0.5)
    label_text = f"Bias (Surface): {surface_bias:.2f}\nRMSE (Surface): {surface_rmse:.2f}\nMAE (Surface): {surface_mae:.2f}\nIOA (Surface): {surface_ioa:.2f}\nBias (Bottom): {bottom_bias:.2f}\nRMSE (Bottom): {bottom_rmse:.2f}\nMAE (Bottom): {bottom_mae:.2f}\nIOA (Bottom): {bottom_ioa:.2f}"
    axs[i].text(1.02, 0.6, label_text, transform=axs[i].transAxes, fontsize=7, bbox=bbox_props)
  
    # set y-axis limits and ticks, and add y-axis label
    axs[i].set_ylim([14, 28])
    axs[i].set_yticks([16, 18, 20, 22, 24, 26, 28])
    axs[i].set_ylabel('TEMP (°C)')
    
    # set subplot title and x-axis label to date format
    axs[i].set_title(site)
    axs[i].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
    axs[i].tick_params(axis='x', rotation=45)

    # Add a title to the stat label box
    axs[i].text(1.065, 1.02, "Model Stats", fontsize=7, fontweight='bold', transform=axs[i].transAxes, ha='center')

# add common legend under the last subplot
handles, labels = axs[2].get_legend_handles_labels()
fig.legend(handles, labels, loc='lower center', ncol=4)

# adjust subplot spacing and save the figure
fig.subplots_adjust(hspace=0.6)
plt.savefig('figures/TEMP/temp_sites1_time_series_obs_vs_mod_TS2_with_stats.jpeg')

```


    
![png](Tuflowfv_validation_statsV2_files/Tuflowfv_validation_statsV2_46_0.png)
    



```python

```
