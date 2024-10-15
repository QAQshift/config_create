<template>
  <div>
    <!-- 配置项选择和操作 -->
    <el-button type="primary" @click="showDialog = true">添加配置项</el-button>

    <el-dialog v-model="showDialog" title="添加配置项">
      <el-form>
        <el-form-item label="国家">
          <el-select v-model="newItem.country" placeholder="请选择国家" :disabled="isCountrySelected"
            @change="onCountryChange">
            <el-option label="AM" value="AM"></el-option>
            <el-option label="MXG" value="MXG"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用户类型">
          <el-select v-model="newItem.userType" placeholder="请选择用户类型" :disabled="isUserTypeSelected"
            @change="onUserTypeChange">
            <el-option label="a" value="a"></el-option>
            <el-option label="b" value="b"></el-option>
          </el-select>
        </el-form-item>
        <div><el-text class="mx-1" type="primary">添加操作</el-text></div>
        <el-form-item label="操作">
          <el-select v-model="selectedAction" placeholder="请选择操作">
            <el-option label="发送文字内容" value="sendMessage"></el-option>
            <el-option label="发送文件" value="sendFile"></el-option>
            <el-option label="发送视频图片" value="sendPicture"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="参数">
          <el-input v-model="actionParam" placeholder="输入操作参数" :class="{ 'is-error': actionParamError }"></el-input>
          <span v-if="actionParamError" class="error-text">参数不能为空</span>
        </el-form-item>
        <el-button type="primary" @click="addAction">添加操作</el-button>

        <!-- 显示已添加的操作 -->
        <div>
          <div>已添加的操作</div>
          <div v-for="(action, idx) in newItem.actions" :key="idx">
            <div>idx</div>
            <div>
              <span>操作:</span><el-tag type="primary">{{ action.func }}</el-tag>
            </div>
            <div></div>
            <span>参数:</span><el-tag type="primary">{{ action.params.join(', ') }}</el-tag>
          </div>
        </div>

        <el-button type="success" @click="addConfigItem">添加配置项</el-button>
      </el-form>
    </el-dialog>

    <!-- 显示已添加配置项 -->
    <el-collapse>
      <el-collapse-item v-for="(userTypes, country) in config" :key="country">
        <template #title>
          {{ country }}
        </template>
        <div v-for="(actions, userType) in userTypes" :key="userType">
          <h4>用户类型: {{ userType }}</h4>
          <div v-for="(action, idx) in actions" :key="idx">
            操作: {{ action.func }} 参数: {{ action.params.join(', ') }}
          </div>
        </div>
      </el-collapse-item>
    </el-collapse>

    <!-- 使用 Element Plus 的文件上传组件 -->
    <el-upload ref="upload" :drag="true" action="#" :limit="1" :on-exceed="handleExceed" :before-upload="handleUpload" accept=".js"
      :on-change="onUploadChange" :auto-upload="false" :on-remove="onRemoveChange">
      <!-- <el-button slot="trigger" type="primary">上传模板文件</el-button> -->
      <div slot="tip" class="el-upload__tip">请选择一个 JavaScript (.js) 模板文件。</div>
    </el-upload>

    <!-- 合并配置项并插入模板文件 -->
    <el-button type="primary" @click="insertConfigIntoTemplate" :disabled="!templateContent">合并配置项到模板并导出</el-button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
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
const actionParam = ref<string>('');
const actionParamError = ref(false);
const config = ref<Config>({});
const templateContent = ref<string | null>(null);
const mergedContent = ref<string | null>(null);

// 下拉选择禁用状态
const isCountrySelected = ref(false);
const isUserTypeSelected = ref(false);

const onCountryChange = () => {
  console.log('newItem.value', newItem.value)
  if (newItem.value.country) {
    console.log("国家禁用")
    isCountrySelected.value = true; // 禁用国家选择
  }
};

const onUserTypeChange = () => {

  if (newItem.value.userType) {
    console.log("用户类型禁用")
    isUserTypeSelected.value = true; // 禁用用户类型选择
  }
};

const addAction = () => {
  if (!actionParam.value) {
    actionParamError.value = true; // 参数必填校验
  } else {
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
    isCountrySelected.value = false;
    isUserTypeSelected.value = false;
    showDialog.value = false; // 关闭对话框
  }
};

// 使用 Element Plus 的上传组件，自定义上传逻辑
const handleUpload = (uploadFile: any) => {
  console.log(123)
  const file = uploadFile.file;
  const reader = new FileReader();

  reader.onload = (e) => {
    templateContent.value = e.target?.result as string;
    console.log(templateContent.value)
  };
  reader.readAsText(file);
};

// 插入配置到模板文件
const insertConfigIntoTemplate = () => {
  if (templateContent.value) {
    if(JSON.stringify(config.value)=="{}"){
      ElMessage({
        type: 'warning',
        message: '请添加配置项',
      })
      return
    }
    const configString = `const config = ${JSON.stringify(config.value, null, 2)};`;
    mergedContent.value = templateContent.value.replace('// INSERT_CONFIG_HERE', configString);
    const blob = new Blob([mergedContent.value], { type: 'application/javascript' });
    saveAs(blob, 'mergedConfig.js');

    ElMessage({
        type: 'success',
        message: '导出成功',
      })
  }else{
    ElMessage({
        type: 'warning',
        message: '请上传模板',
      })
  }

};

const onUploadChange = (uploadFile: UploadFile) => {
  console.log("change")
  const file = uploadFile.raw;
  const reader = new FileReader();

  reader.onload = (e) => {
    templateContent.value = e.target?.result as string;
    // console.log(templateContent.value)
  };
  reader.readAsText(file as UploadRawFile);
}

const onRemoveChange = () => {
  console.log("移除")
  templateContent.value=null
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
</style>
