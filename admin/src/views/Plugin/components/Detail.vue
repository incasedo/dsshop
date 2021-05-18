<template>
  <div v-loading="loading" class="container">
    <el-form ref="ruleForm" :model="ruleForm" :rules="rules" label-width="100px" class="ruleForm">
      <h3>基本信息</h3>
      <el-form-item class="min-input" label="插件名称" prop="name">
        <el-input v-model="ruleForm.name" maxlength="20" placeholder="请输入插件名称" clearable/>
      </el-form-item>
      <el-form-item class="min-input" label="插件标识" prop="abbreviation">
        <el-input v-model="ruleForm.abbreviation" maxlength="20" placeholder="请输入插件标识" clearable/>
      </el-form-item>
      <el-form-item class="min-input" label="作者" prop="author">
        <el-input v-model="ruleForm.author" maxlength="20" placeholder="请输入作者" clearable/>
      </el-form-item>
      <el-form-item class="min-input" label="插件简介" prop="describe">
        <el-input v-model="ruleForm.describe" maxlength="200" placeholder="请输入插件简介" clearable/>
      </el-form-item>
      <el-form-item class="min-input" label="插件版本" prop="versions">
        <el-input v-model="ruleForm.versions" maxlength="20" placeholder="请输入插件版本" clearable/>
      </el-form-item>
      <h3>数据库</h3>
      <div class="tip">
        <p>1、创建数据表后将会自动生成数据库迁移文件、控制器、模型、验证器，如有不需要的文件，请在生成后进行删除</p>
        <p>2、删除插件时，将会删除自动生成的文件，其它非自动生成的文件不会进行删除</p>
        <p>3、在未发布时，修改数据库可以直接修改当前版本下的数据库迁移文件，当已经发布后，会自动生成修改的数据迁移文件</p>
      </div>
      <el-table
        :data="ruleForm.db"
        style="width: 100%"
        height="250">
        <el-table-column
          prop="name"
          label="表名"
          width="200">
          <template slot-scope="scope">
            {{ scope.row.name }}
          </template>
        </el-table-column>
        <el-table-column
          prop="softDeletes"
          label="软删除"
          width="200">
          <template slot-scope="scope">
            {{ scope.row.annotation ? '支持' : '不支持' }}
          </template>
        </el-table-column>
        <el-table-column
          prop="timestamps"
          label="timestamps"
          width="200">
          <template slot-scope="scope">
            {{ scope.row.timestamps ? '支持' : '不支持' }}
          </template>
        </el-table-column>
        <el-table-column
          prop="annotation"
          label="表注释">
          <template slot-scope="scope">
            {{ scope.row.annotation }}
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="120"
          fixed="right">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="dark" content="编辑" placement="top-start">
              <el-button type="primary" icon="el-icon-edit" circle @click="editDataTable(scope.row)"/>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="删除" placement="top-start">
              <el-button type="danger" icon="el-icon-delete" circle @click="deleteDataTable(scope.$index)"/>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-form-item v-if="name" class="min-input" label="是否重置" prop="reset">
        <el-switch
          v-model="ruleForm.reset"
          active-text="是"
          inactive-text="否"/>
      </el-form-item>
      <div class="tip">
        <p>1、勾选重置将重新生成所有插件相关文件，如已对部分文件做了修改，请不要勾选</p>
        <p>2、不重置的话，保存后会不将修改后的数据更新到对应的文件中，需要自行在对应文件中手动添加</p>
      </div>
      <el-button style="margin: 10px 0 50px 0;" type="success" round @click="addDataTable">新建数据表</el-button>
      <el-form-item class="float-button">
        <el-button :loading="formLoading" type="primary" @click="submit">提交</el-button>
      </el-form-item>
    </el-form>
    <!--新建数据表-->
    <el-dialog :close-on-click-modal="false" :visible.sync="dialogDataTable" :fullscreen="true" title="新建数据表">
      <el-form ref="dataTableForm" :model="temp" :rules="dataTableRules" class="dataTableForm" label-position="top" label-width="120px">
        <el-form-item class="min-input" label="表名" prop="name">
          <el-input v-model="temp.name" placeholder="请输入表名" maxlength="60" clearable/>
        </el-form-item>
        <el-form-item class="min-input" label="表注释" prop="annotation">
          <el-input v-model="temp.annotation" placeholder="请输入表注释" maxlength="60" clearable/>
        </el-form-item>
        <el-form-item label="属性" prop="attribute">
          <el-table
            :data="temp.attribute"
            style="width: 100%"
            height="250">
            <el-table-column
              prop="name"
              label="名字"
              width="200">
              <template slot-scope="scope">
                <el-form-item
                  :rules="{
                    required: true, message: '属性名字不能为空', trigger: 'blur'
                  }"
                  :prop="'attribute.'+scope.$index+'.name'"
                >
                  <el-input v-model="scope.row.name" clearable/>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="type"
              label="类型"
              width="300">
              <template slot-scope="scope">
                <el-form-item
                  :rules="{
                    required: true, message: '请选择类型', trigger: 'change'
                  }"
                  :prop="'attribute.'+scope.$index+'.type'"
                >
                  <el-select v-model="scope.row.type" placeholder="请选择" filterable clearable>
                    <el-option-group
                      v-for="group in db.type"
                      :key="group.label"
                      :label="group.label">
                      <el-option
                        v-for="item in group.options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"/>
                    </el-option-group>
                  </el-select>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="length"
              label="长度/值"
              width="120">
              <template slot-scope="scope">
                <el-form-item
                  :prop="'attribute.'+scope.$index+'.length'"
                >
                  <el-input v-model="scope.row.length" clearable/>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="default"
              label="默认值"
              width="200">
              <template slot-scope="scope">
                <el-form :model="scope.row">
                  <el-form-item prop="default">
                    <el-input v-model="scope.row.default" clearable/>
                  </el-form-item>
                </el-form>
              </template>
            </el-table-column>
            <el-table-column
              prop="sort"
              label="排序规则"
              width="300">
              <template slot-scope="scope">
                <el-form-item
                  :rules="{
                    required: true, message: '请选择排序规则', trigger: 'change'
                  }"
                  :prop="'attribute.'+scope.$index+'.sort'"
                >
                  <el-select v-model="scope.row.sort" placeholder="请选择" filterable clearable>
                    <el-option
                      v-for="item in db.sort"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"/>
                  </el-select>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="attribute"
              label="属性"
              width="300">
              <template slot-scope="scope">
                <el-form-item>
                  <el-select v-model="scope.row.attribute" placeholder="请选择" filterable clearable>
                    <el-option
                      v-for="item in db.attribute"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"/>
                  </el-select>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="is_empty"
              label="空"
              width="45">
              <template slot-scope="scope">
                <el-form-item prop="is_empty">
                  <el-checkbox v-model="scope.row.is_empty"/>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="AUTO_INCREMENT"
              label="A_I"
              width="45">
              <template slot-scope="scope">
                <el-form-item prop="AUTO_INCREMENT">
                  <el-checkbox v-model="scope.row.AUTO_INCREMENT"/>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              prop="annotation"
              label="注释">
              <template slot-scope="scope">
                <el-form-item prop="annotation">
                  <el-input v-model="scope.row.annotation" clearable/>
                </el-form-item>
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              width="50"
              fixed="right">
              <template slot-scope="scope">
                <el-tooltip class="item" effect="dark" content="删除" placement="top-start">
                  <el-button type="danger" icon="el-icon-delete" circle @click="deleteDBAttribute(scope.$index)"/>
                </el-tooltip>
              </template>
            </el-table-column>
          </el-table>
          <el-button style="margin-top:10px;" type="success" round @click="addDBAttribute">增加属性</el-button>
        </el-form-item>
        <el-form-item class="min-input" label="软删除" prop="softDeletes">
          <el-radio-group v-model="temp.softDeletes">
            <el-radio :label="1">支持</el-radio>
            <el-radio :label="0">不支持</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item class="min-input" label="timestamps" prop="timestamps">
          <el-radio-group v-model="temp.timestamps">
            <el-radio :label="1">支持</el-radio>
            <el-radio :label="0">不支持</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="索引" prop="indexes">
          <el-table
            :data="temp.indexes"
            style="width: 100%"
            height="250">
            <el-table-column
              prop="name"
              label="键名"
              width="200">
              <template slot-scope="scope">
                {{ scope.row.name }}
              </template>
            </el-table-column>
            <el-table-column
              prop="type"
              label="索引类型"
              width="200">
              <template slot-scope="scope">
                {{ scope.row.type }}
              </template>
            </el-table-column>
            <el-table-column
              prop="field"
              label="字段">
              <template slot-scope="scope">
                {{ scope.row.field }}
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              width="120"
              fixed="right">
              <template slot-scope="scope">
                <el-tooltip class="item" effect="dark" content="编辑" placement="top-start">
                  <el-button type="primary" icon="el-icon-edit" circle @click="editIndexes(scope.row)"/>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="删除" placement="top-start">
                  <el-button type="danger" icon="el-icon-delete" circle @click="deleteIndexes(scope.$index)"/>
                </el-tooltip>
              </template>
            </el-table-column>
          </el-table>
          <el-button style="margin-top:10px;" type="success" round @click="addDBIndexes">增加索引</el-button>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button :loading="formLoading" @click="dialogDataTable = false">{{ $t('usuel.cancel') }}</el-button>
        <el-button :loading="formLoading" type="primary" @click="dataTableSubmit">确定</el-button>
      </div>
    </el-dialog>
    <!-- 添加索引-->
    <el-dialog :close-on-click-modal="false" :visible.sync="dialogIndexes" :title="dialogIndexesIndex === '' ? `新建索引` : `修改索引`">
      <el-form ref="indexesForm" :model="indexesTemp" :rules="indexesRules" label-position="left" label-width="120px">
        <el-form-item label="索引名称" prop="name">
          <el-input v-model="indexesTemp.name" class="min-input" placeholder="请输入索引名称" maxlength="60" clearable/>
        </el-form-item>
        <el-form-item label="索引类型" prop="type">
          <el-select v-model="indexesTemp.type" placeholder="请选择" clearable>
            <el-option
              v-for="item in indexesType"
              :key="item.value"
              :label="item.label"
              :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item label="字段" prop="field">
          <el-select v-model="indexesTemp.field" multiple placeholder="请选择" clearable>
            <el-option
              v-for="item in temp.attribute"
              :key="item.name"
              :label="item.name"
              :value="item.name"/>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button :loading="formLoading" @click="dialogIndexes = false">{{ $t('usuel.cancel') }}</el-button>
        <el-button :loading="formLoading" type="primary" @click="indexesSubmit">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<style lang='scss' scoped>
  @import "../scss/detail";
</style>
<script>
import detail from '../js/detail'
export default detail
</script>