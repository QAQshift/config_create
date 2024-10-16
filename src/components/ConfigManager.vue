<template>
  <div>
    <!-- 配置项选择和操作 -->
    <el-button type="primary" circle @click="showDialog = true"
      :style="{ width: '20vw', height: '20vw', marginBottom: '20px', fontSize: '2vw' }">添加配置项</el-button>



    <el-dialog v-model="showDialog" title="添加配置项" @closed="dialogClosed">
      <el-form>
        <el-form-item label="国家">
          <el-input v-model="newItem.country" placeholder="请输入国家前缀"
            :class="{ 'is-error': countryParamError }" :disabled="isCountryInputed"></el-input>
        </el-form-item>
        <el-form-item label="用户类型">
          <el-input v-model="newItem.userType" placeholder="请输入用户类型前缀"
            :class="{ 'is-error': userTypeParamError }" :disabled="isUserTypeInputed"></el-input>
        </el-form-item>
        <div :style="{ marginBottom: '18px' }"><el-text class="mx-1" type="primary">添加操作</el-text></div>
        <el-form-item label="操作">
          <el-select v-model="selectedAction" placeholder="请选择操作" @change="() => {
            isSelectAction = true
          }">
            <el-option label="发送文字内容" value="sendMessage"></el-option>
            <el-option label="发送文件" value="sendFile"></el-option>
            <el-option label="发送视频图片" value="sendPicture"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="customLabel" v-show="isSelectAction">
          <el-input v-model="actionParam" placeholder="输入对应的内容" :class="{ 'is-error': actionParamError }"></el-input>
          <span v-if="actionParamError" class="error-text">参数不能为空</span>
        </el-form-item>
        <div :style="{ display: 'flex', justifyContent: 'flex-end' }">
          <el-button type="primary" @click="addAction">添加操作</el-button>
        </div>
        <el-divider></el-divider>

        <!-- 显示已添加的操作 -->
        <div :style="{ display: 'flex', flexDirection: 'column', width: '100%' }">
          <div :style="{ fontSize: '18px', textAlign: 'left' }">已添加的操作</div>
          <div v-for="(action, idx) in newItem.actions" :key="idx"
            :style="{ padding: '10px 20px', boxSizing: 'border-box', display: 'flex' }">
            <div :style="{ paddingRight: '10px' }">
              {{ idx + 1 }} ---
            </div>
            <div :style="{ display: 'flex', textAlign: 'left', flexDirection: 'column', flex: 1 }">
              <div><span> 操作：</span>
                <el-tag type="primary">{{ action.func == 'sendMessage' ? '发送消息 ' : action.func == 'sendFile' ? '发送文件 ' :
                  '发送图片视频 ' }}</el-tag>
              </div>
              <div :style="{ padding: '6px 0' }">
                <div>{{ action.func == 'sendMessage' ? '消息内容 ' : action.func == 'sendFile' ? '文件名 ' : '相册名 ' }}：</div>
                <div class="info_content">
                  {{
                    action.params.join(',')
                  }}</div>
              </div>
            </div>

          </div>

        </div>

        <el-button type="success" @click="addConfigItem">添加配置项</el-button>
      </el-form>
    </el-dialog>

    <!-- 显示已添加配置项 -->
    <el-collapse :style="{ width: '50vw', marginBottom: '20px' }">
      <el-collapse-item v-for="(userTypes, country) in config" :key="country">
        <template #title>
          {{ country }}
        </template>
        <div v-for="(actions, userType) in userTypes" :key="userType"
          :style="{ display: 'flex', flexDirection: 'column', textAlign: 'left' }">
          <div>用户类型：<el-tag type="primary">{{ userType }}</el-tag> </div>
          <div v-for="(action, idx) in actions" :key="idx" :style="{ padding: '10px 40px' }">
            <div>
              <span>操作：</span>
              <el-tag type="primary">{{ action.func == 'sendMessage' ? '发送消息 ' : action.func == 'sendFile' ? '发送文件 ' :
                '发送图片视频 ' }}</el-tag>
            </div>
            <div :style="{ padding: '6px 0' }">
              <div>{{ action.func == 'sendMessage' ? '消息内容 ' : action.func == 'sendFile' ? '文件名 ' : '相册名 ' }}：</div>
              <div class="info_content">
                {{
                  action.params.join(',')
                }}</div>
            </div>
            <div :style="{ display: 'flex', justifyContent: 'flex-end' }">
              <el-button type="danger" @click="deleteUserType(String(country), String(userType), idx)">删除</el-button>
            </div>
          </div>
        </div>
      </el-collapse-item>
    </el-collapse>

    <!-- 使用 Element Plus 的文件上传组件 -->

    <el-upload ref="upload" :drag="true" action="#" :limit="1" :on-exceed="handleExceed" accept=".js"
      :on-change="onUploadChange" :auto-upload="false" :on-remove="onRemoveChange">

      <div class="el-upload__tip">请选择一个 JavaScript (.js) 模板文件。</div>

    </el-upload>



    <!-- 合并配置项并插入模板文件 -->
    <div :style="{ display: 'flex', flexDirection: 'column', alignItems: 'center' }">
      <el-button type="primary" @click="insertConfigIntoTemplate" :disabled="!templateContent">导出</el-button>
      <span :style="{ color: '#67c23a', fontSize: '12px', padding: '5px' }">合并配置项到模板并导出</span>
    </div>


  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { saveAs } from 'file-saver';
import { ElMessage, genFileId, UploadFile, UploadInstance, UploadProps, UploadRawFile } from 'element-plus';

// 定义操作的类型
interface Action {
  func: 'sendMessage' | 'sendFile' | 'sendPicture';
  params: string[];
}

// 定义配置类型
interface Config {
  [country: string]: {
    [userType: string]: Action[];
  };
}

const showDialog = ref(false);
const newItem = ref<{ country: string; userType: string; actions: Action[] }>({ country: '', userType: '', actions: [] });
const selectedAction = ref<string>('');
const isSelectAction = ref<boolean>(false)
const actionParam = ref<string>('');
const countryParamError = ref(false);
const userTypeParamError = ref(false);
const actionParamError = ref(false);
const config = ref<Config>({});
const templateContent = ref<string | null>(null);
const mergedContent = ref<string | null>(null);
const filename = ref<string>('')

// 下拉选择禁用状态
const isCountryInputed = ref(false);
const isUserTypeInputed = ref(false);

const customLabel = computed(() => {
  if (selectedAction.value === 'sendMessage') {
    return '消息内容 ';
  } else if (selectedAction.value === 'sendFile') {
    return '文件名 ';
  } else {
    return '相册名 ';
  }
});


const addAction = () => {
  if (!actionParam.value) {
    actionParamError.value = true; // 参数必填校验
  }
  if (!newItem.value.country) {
    countryParamError.value = true; // 参数必填校验
  }
  if (!newItem.value.userType) {
    userTypeParamError.value = true; // 参数必填校验
  }

  if (actionParam.value && newItem.value.country && newItem.value.userType) {
    // 添加动作的时候禁用国家和用户 输入
    isCountryInputed.value = true;
    isUserTypeInputed.value = true;


    countryParamError.value = false;
    userTypeParamError.value = false;
    actionParamError.value = false;
    
    newItem.value.actions.push({
      func: selectedAction.value as 'sendMessage' | 'sendFile' | 'sendPicture',
      params: [actionParam.value]
    });
    // 清空操作选择和参数输入
    selectedAction.value = '';
    actionParam.value = '';
  }
};

const dialogClosed = () => {
  isCountryInputed.value = false;
  isUserTypeInputed.value = false;
  newItem.value = { country: '', userType: '', actions: [] }
}

const addConfigItem = () => {
  const { country, userType, actions } = newItem.value;
  if (country && userType && actions.length > 0) {
    if (!config.value[country]) {
      config.value[country] = {};
    }
    if (!config.value[country][userType]) {
      config.value[country][userType] = [];
    }
    config.value[country][userType].push(...actions);
    // 重置对话框内容和禁用状态
    newItem.value = { country: '', userType: '', actions: [] };
    isCountryInputed.value = false;
    isUserTypeInputed.value = false;
    showDialog.value = false; // 关闭对话框
  } else {
    ElMessage({
      message: '请确保国家和用户类型都已选择，且操作已经添加.',
      type: 'warning',
    })
  }
};

const deleteUserType = (country: string, userType: string, actionIndex: number) => {
  // 检查 country 是否存在于 config 对象中
  if (config.value[country]) {
    // 检查 userType 是否存在于 country 下
    if (config.value[country][userType]) {
      // 删除该 userType 下的特定 action 项
      config.value[country][userType].splice(actionIndex, 1);

      // 如果 userType 下的 actions 已经空了，删除该 userType
      if (config.value[country][userType].length === 0) {
        delete config.value[country][userType];
      }

      // 如果 country 下所有 userType 都被删除了，删除 country
      if (Object.keys(config.value[country]).length === 0) {
        delete config.value[country];
      }
    }
  }
};

// 插入配置到模板文件
const insertConfigIntoTemplate = () => {
  if (templateContent.value) {
    if (JSON.stringify(config.value) == "{}") {
      ElMessage({
        type: 'warning',
        message: '请添加配置项',
      })
      return
    }
    const configString = `const config = ${JSON.stringify(config.value, null, 2)};`;
    mergedContent.value = templateContent.value.replace('// INSERT_CONFIG_HERE', configString);
    const blob = new Blob([mergedContent.value], { type: 'application/javascript' });
    saveAs(blob, filename.value);

    ElMessage({
      type: 'success',
      message: '导出成功',
    })
  } else {
    ElMessage({
      type: 'warning',
      message: '请上传模板',
    })
  }

};

const onUploadChange = (uploadFile: UploadFile) => {

  const file = uploadFile.raw;
  filename.value = uploadFile.name.replace('模板', '')
  console.log("change", filename.value)
  const reader = new FileReader();

  reader.onload = (e) => {
    templateContent.value = e.target?.result as string;
    // console.log(templateContent.value)
  };
  reader.readAsText(file as UploadRawFile);
}

const onRemoveChange = () => {
  console.log("移除")
  templateContent.value = null
}


const upload = ref<UploadInstance>()
const handleExceed: UploadProps['onExceed'] = (files) => {
  upload.value!.clearFiles()
  const file = files[0] as UploadRawFile
  file.uid = genFileId()
  upload.value!.handleStart(file)
}
</script>

<style>
.error-text {
  color: red;
  font-size: 12px;
}

.is-error {
  border-color: red;
}

.info_content {
  color: #909399;
  background-color: rgb(243.9, 244.2, 244.8);
  padding: 4px 8px;
  border-radius: 4px;
  border: 1px solid rgb(232.8, 233.4, 234.6);
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-top: 6px;
}
</style>
