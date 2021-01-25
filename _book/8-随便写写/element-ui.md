#element-ui 坑的记录
    `安装过程看官网,我采用的是按需引入`

### element-ui Form  表单组件 下拉框选择与显示不符合

```
选择的options与实际点击的结果不符
el-select显示正确的选择条件是

el-select 的v-model 和 el-option 的value 必须是单一项，不能是数组或对象
<el-form ref="form" label-width="80px">
    <el-select v-model="cityName" placeholder="请选择城市">
        <el-option label="北京" value="beijing" key="id"></el-option>
    </el-select>
</el-form>

```
