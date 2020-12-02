### 查询网址

(https://browserl.ist/)

### Browserslist配合的插件
```
Autoprefixer
Babel
postcss-preset-env
eslint-plugin-compat
stylelint-no-unsupported-browser-features
postcss-normalize
```

### 浏览器列表

```
您可以通过查询指定浏览器和Node.js版本（不区分大小写）：
> 5%：全球使用情况统计选择的浏览器版本。 >=，<也<=工作。
> 5% in US：使用美国使用情况统计。它接受两个字母的国家/地区代码。
> 5% in alt-AS：使用亚洲地区使用情况统计。可在以下位置找到所有地区代码的列表caniuse-lite/data/regions。
> 5% in my stats：使用自定义使用数据。
cover 99.5%：提供覆盖的最流行的浏览器。
cover 99.5% in US：与上述相同，使用双字母国家代码。
cover 99.5% in my stats：使用自定义使用数据。
maintained node versions：所有Node.js版本，仍由 Node.js Foundation 维护。
node 10和node 10.4：选择最新的Node.js 10.x.x 或10.4.x发布。
current node：Browserslist目前使用的Node.js版本。
extends browserslist-config-mycompany：从browserslist-config-mycompanynpm包中获取查询 。
ie 6-8：选择包含范围的版本。
Firefox > 20：Firefox的版本比20更新 >=，<并且也可以<=工作。
iOS 7：iOS浏览器版本7直接。
Firefox ESR：最新的[Firefox ESR]版本。
unreleased versions或unreleased Chrome versions：alpha和beta版本。
last 2 major versions或last 2 iOS major versions：最近2个主要版本的所有次要/补丁版本。
since 2015或last 2 years：自2015年以来发布的所有版本（也since 2015-03和since 2015-03-10）。
dead：来自last 2 version查询的浏览器，但全球使用统计数据少于0.5％，且24个月内没有官方支持或更新。现在是IE 10，IE_Mob 10，BlackBerry 10， BlackBerry 7，和OperaMobile 12.1。
last 2 versions：每个浏览器的最后两个版本。
last 2 Chrome versions：Chrome浏览器的最后两个版本。
defaults：Browserslist的默认浏览器（> 0.5%, last 2 versions, Firefox ESR, not dead）。
not ie <= 8：排除先前查询选择的浏览器。

```