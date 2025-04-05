<template>
  <div class="area-selection-container">
    <div class="horizontal-layout">
      <!-- 抽测范围配置 -->
      <div class="config-panel">
        <div class="panel-header">
          <Icon type="ios-information-circle-outline" />
          <span>抽测范围配置</span>
        </div>
        <div class="panel-content">
          <Form :label-width="80" ref="formValidate">
            <FormItem label="抽取项目" required>
              <Input v-model="selectedItem" placeholder="请输入抽取项目" />
            </FormItem>
            <FormItem label="抽取人" required>
              <Input v-model="operator" placeholder="请输入抽取人" />
            </FormItem>
            <FormItem label="抽选原则" required>
              <Select v-model="selectedPrinciple" placeholder="随机抽取">
                <Option value="随机抽取" label="随机抽取"></Option>
              </Select>
            </FormItem>
            <FormItem label="抽选数量" required>
              <Input v-model="selectCount" placeholder="请输入抽选数量" />
            </FormItem>
            <FormItem label="高层" class="high-level-item">
              <div class="high-level-content">
                <div class="checkbox-row">
                  <Checkbox v-model="buildingSelected">楼栋</Checkbox>
                  <Checkbox v-model="floorSelected">层</Checkbox>
                  <Checkbox v-model="roomSelected">户</Checkbox>
                </div>
                <div v-for="(group, index) in buildingGroups" :key="index" class="building-group">
                  <div class="input-row">
                    <FormItem 
                      :prop="'buildingGroups.' + index + '.building'"
                      style="margin-bottom: 0;"
                      :error="errorMessages[index] && errorMessages[index].building"
                    >
                      <Input 
                        v-model="group.building" 
                        placeholder="请输入栋" 
                        :disabled="!buildingSelected"
                        @on-blur="validateAndShowError(group.building, 'building', index)"
                      >
                        <span slot="prepend">栋</span>
                        <span slot="append"><Icon type="ios-search" /></span>
                      </Input>
                    </FormItem>
                    
                    <FormItem 
                      v-if="floorSelected"
                      :prop="'buildingGroups.' + index + '.floor'"
                      style="margin-bottom: 0;"
                      :error="errorMessages[index] && errorMessages[index].floor"
                    >
                      <Input 
                        v-model="group.floor" 
                        placeholder="请输入楼层" 
                        :disabled="!floorSelected"
                        @on-blur="validateAndShowError(group.floor, 'floor', index)"
                      >
                        <span slot="prepend">层</span>
                        <span slot="append"><Icon type="ios-search" /></span>
                      </Input>
                    </FormItem>
                    
                    <FormItem 
                      v-if="roomSelected"
                      :prop="'buildingGroups.' + index + '.room'"
                      style="margin-bottom: 0;"
                      :error="errorMessages[index] && errorMessages[index].room"
                    >
                      <Input 
                        v-model="group.room" 
                        placeholder="请输入户" 
                        :disabled="!roomSelected"
                        @on-blur="validateAndShowError(group.room, 'room', index)"
                      >
                        <span slot="prepend">户</span>
                        <span slot="append"><Icon type="ios-search" /></span>
                      </Input>
                    </FormItem>
                    
                    <Button 
                      v-if="index === buildingGroups.length - 1"
                      type="default" 
                      shape="circle" 
                      icon="md-add"
                      @click="addBuildingGroup"
                    ></Button>
                    <Button 
                      v-else
                      type="error" 
                      shape="circle" 
                      icon="md-close"
                      @click="removeBuildingGroup(index)"
                    ></Button>
                  </div>
                </div>
                <div class="symbol-input">
                  <span>规则：</span>
                  <span>栋号仅支持单个数字，楼层和户号支持：单个数字(1)，范围(3-4)，多个范围(3-4,5-8,10-12)</span>
                </div>
              </div>
            </FormItem>
          </Form>
          <!-- <div class="add-new-type">
            <Icon type="md-add-circle" />
            <span>新增类型</span>
          </div> -->
          <div class="action-buttons">
            <Button type="default">取 消</Button>
            <Button type="primary" @click="handleSubmit" :disabled="!isFormValid">保 存</Button>
          </div>
        </div>
      </div>

      <!-- 抽测范围列表 -->
      <div class="result-panel">
        <div class="panel-header">
          <span>抽测范围列表</span>
        </div>
        <div class="panel-content">
          <Table border :columns="columns" :data="tableData" empty-text="暂无数据，请在左侧'配置抽测范围'">
          </Table>
          <div class="table-actions">
            <Button type="default">清空列表</Button>
            <Button type="primary">统一抽选</Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 抽取记录 -->
    <div class="record-panel">
      <div class="panel-header">
        <span>抽取记录</span>
      </div>
      <div class="panel-content">
        <Table border :columns="recordColumns" :data="recordData">
        </Table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AreaSelection',
  data() {
    return {
      // 抽测范围配置
      selectedItem: '测试项目1',
      selectedPrinciple: '随机抽取',
      selectCount: '3',
      operator: '陈丽',
      
      // 高层配置
      buildingSelected: true,
      floorSelected: true,
      roomSelected: true,

      // 新增楼栋组数据
      buildingGroups: [{
        building: '',
        floor: '',
        room: ''
      }],

      // 抽测范围列表
      columns: [
        { type: 'selection', width: 60, align: 'center' },
        { title: '抽取项目', key: 'item' },
        { title: '抽取范围', key: 'range' },
        { title: '抽取原则', key: 'principle' },
        { title: '抽测数量', key: 'count' },
        { title: '操作', key: 'action', width: 150, render: (h, params) => {
          return h('div', [
            h('Button', {
              props: {
                type: 'primary',
                size: 'small'
              },
              style: {
                marginRight: '5px'
              },
              on: {
                click: () => {
                  this.operate(params.index)
                }
              }
            }, '操作')
          ])
        }}
      ],
      tableData: [],
      
      // 抽取记录
      recordColumns: [
        { title: '记录组', key: 'group', width: 100 },
        { title: '抽取项目', key: 'item' },
        { title: '状态', key: 'status', width: 120, render: (h, params) => {
          return h('div', [
            h('Tag', {
              props: {
                color: params.row.status === '正常' ? 'success' : 'error'
              }
            }, params.row.status)
          ])
        }},
        { title: '作废说明', key: 'description' },
        { title: '抽取人', key: 'operator' },
        { title: '抽测时间', key: 'time' },
        { title: '操作', key: 'action', width: 200, render: (h, params) => {
          return h('div', [
            h('Button', {
              props: {
                type: 'primary',
                size: 'small'
              },
              style: {
                marginRight: '5px'
              }
            }, '结果展示'),
            h('Button', {
              props: {
                size: 'small'
              },
              style: {
                marginRight: '5px'
              }
            }, '复制'),
            h('Button', {
              props: {
                type: 'error',
                size: 'small'
              }
            }, '作废')
          ])
        }}
      ],
      recordData: [
        {
          group: '第1组',
          item: '实测实量（全阶段）',
          status: '正常',
          description: '',
          operator: '王玉柔',
          time: '2023-03-26 14:16:01'
        }
      ],

      // 新增: 错误消息存储
      errorMessages: [],
    }
  },
  created() {
    // 初始化错误消息数组
    this.initErrorMessages();
    // 初始化表格数据
    this.initTableData();
  },
  methods: {
    // 初始化错误消息数组
    initErrorMessages() {
      this.errorMessages = this.buildingGroups.map(() => ({
        building: '',
        floor: '',
        room: ''
      }));
    },
    
    // 添加楼栋组时同步添加错误消息对象
    addBuildingGroup() {
      this.buildingGroups.push({
        building: '',
        floor: '',
        room: ''
      });
      
      // 同步添加错误消息对象
      this.errorMessages.push({
        building: '',
        floor: '',
        room: ''
      });
    },
    
    // 移除楼栋组时同步移除错误消息对象
    removeBuildingGroup(index) {
      this.buildingGroups.splice(index, 1);
      this.errorMessages.splice(index, 1);
    },
    
    // 更新验证方法，返回具体错误信息
    validateFormat(value, type) {
      // 如果是空值
      if (!value || value === '') {
        return { valid: true, message: '' };
      }
      
      // 转为字符串并处理空格
      value = String(value).replace(/\s+/g, '');
      
      // 栋号验证：只允许单个数字
      if (type === 'building') {
        if (!/^\d+$/.test(value)) {
          return { 
            valid: false, 
            message: '栋号只支持单个数字，如：1' 
          };
        }
        return { valid: true, message: '' };
      }
      
      // 层号和户号验证
      try {
        // 处理中文符号
        value = value.replace(/，/g, ',').replace(/[~～]/g, '-');
        
        // 检查单个数字
        if (/^\d+$/.test(value)) {
          return { valid: true, message: '' };
        }
        
        // 分割逗号部分
        const parts = value.split(',');
        
        // 检查空部分
        if (parts.some(p => p === '')) {
          return {
            valid: false,
            message: '格式错误，不能有空值，如：1,,2'
          };
        }
        
        // 提取所有涉及到的数字，用于检查重复
        const allNumbers = new Set();
        
        for (const part of parts) {
          // 检查是否为单个数字
          if (/^\d+$/.test(part)) {
            const num = parseInt(part, 10);
            
            // 检查重复
            if (allNumbers.has(num)) {
              return {
                valid: false,
                message: `数字 ${num} 重复出现`
              };
            }
            
            allNumbers.add(num);
            continue;
          }
          
          // 检查范围格式
          const range = part.split('-');
          if (range.length === 2) {
            const start = parseInt(range[0], 10);
            const end = parseInt(range[1], 10);
            
            // 检查无效数字
            if (isNaN(start) || isNaN(end)) {
              return {
                valid: false,
                message: `范围格式错误：${part}`
              };
            }
            
            // 检查范围大小关系
            if (start >= end) {
              return {
                valid: false,
                message: `范围必须从小到大：${part}`
              };
            }
            
            // 检查所有数字是否有重复
            for (let i = start; i <= end; i++) {
              if (allNumbers.has(i)) {
                return {
                  valid: false,
                  message: `数字 ${i} 在多个范围中重复出现`
                };
              }
              allNumbers.add(i);
            }
            
            continue;
          }
          
          // 不是有效格式
          return {
            valid: false,
            message: `格式错误：${part}，应为单个数字或范围(如：3-4)`
          };
        }
        
        return { valid: true, message: '' };
      } catch (error) {
        console.error('验证出错:', error);
        return {
          valid: false,
          message: '验证出错，请检查输入格式'
        };
      }
    },
    
    // 更新验证和显示错误的方法
    validateAndShowError(value, type, index) {
      // 确保错误消息数组已初始化
      if (!this.errorMessages[index]) {
        this.errorMessages[index] = {
          building: '',
          floor: '',
          room: ''
        };
      }
      
      // 如果为空，标记为错误（必填）
      if (!value || value === '') {
        this.errorMessages[index][type] = `请输入${type === 'building' ? '栋号' : type === 'floor' ? '楼层' : '户号'}`;
        return false;
      }
      
      // 验证并获取结果
      const result = this.validateFormat(value, type);
      
      // 设置错误消息
      this.errorMessages[index][type] = result.valid ? '' : result.message;
      
      // 如果验证失败，同时显示提示
      if (!result.valid) {
        this.$Message.error(result.message);
      }
      
      return result.valid;
    },
    
    // 更新handleSubmit方法
    handleSubmit() {
      // 手动验证所有输入
      let isValid = true;
      
      // 初始化错误消息
      this.initErrorMessages();
      
      // 验证栋号和其他数据
      this.buildingGroups.forEach((group, index) => {
        // 验证栋号
        const buildingResult = this.validateFormat(group.building, 'building');
        if (!buildingResult.valid) {
          this.errorMessages[index].building = buildingResult.message;
          this.$Message.error(buildingResult.message);
          isValid = false;
        }
        
        // 验证楼层
        if (this.floorSelected && group.floor) {
          const floorResult = this.validateFormat(group.floor, 'floor');
          if (!floorResult.valid) {
            this.errorMessages[index].floor = floorResult.message;
            this.$Message.error(floorResult.message);
            isValid = false;
          }
        }
        
        // 验证户号
        if (this.roomSelected && group.room) {
          const roomResult = this.validateFormat(group.room, 'room');
          if (!roomResult.valid) {
            this.errorMessages[index].room = roomResult.message;
            this.$Message.error(roomResult.message);
            isValid = false;
          }
        }
      });
      
      // 验证全部通过
      if (isValid) {
        // 构建范围字符串
        const buildingRanges = this.buildingGroups.map(group => {
          const parts = [];
          if (this.buildingSelected && group.building) {
            parts.push(`${group.building}栋`);
          }
          if (this.floorSelected && group.floor) {
            parts.push(`${group.floor}层`);
          }
          if (this.roomSelected && group.room) {
            parts.push(`${group.room}户`);
          }
          return parts.join('-');
        }).join('，');

        // 构建新的数据项
        const newItem = {
          item: this.selectedItem,
          range: buildingRanges,
          principle: this.selectedPrinciple,
          count: this.selectCount,
          operator: this.operator
        };

        // 从本地存储获取现有数据
        let existingData = localStorage.getItem('areaSelectionData');
        let dataList = [];

        if (existingData) {
          try {
            dataList = JSON.parse(existingData);
            if (!Array.isArray(dataList)) {
              dataList = [];
            }
          } catch (e) {
            console.error('解析缓存数据出错：', e);
            dataList = [];
          }
        }

        // 添加新数据
        dataList.push(newItem);

        // 保存到本地存储
        localStorage.setItem('areaSelectionData', JSON.stringify(dataList));

        // 更新表格数据
        this.tableData = dataList;

        this.$Message.success('保存成功');

        // 清空表单数据
        this.resetForm();
      } else {
        this.$Message.error('请修正表单中的错误');
      }
    },
    
    // 添加重置表单的方法
    resetForm() {
      this.selectedItem = '';
      this.operator = '';
      this.selectCount = '2';
      this.buildingGroups = [{
        building: '',
        floor: '',
        room: ''
      }];
      this.errorMessages = [{
        building: '',
        floor: '',
        room: ''
      }];
    },

    // 添加初始化数据的方法
    initTableData() {
      const existingData = localStorage.getItem('areaSelectionData');
      if (existingData) {
        try {
          const dataList = JSON.parse(existingData);
          if (Array.isArray(dataList)) {
            this.tableData = dataList;
          }
        } catch (e) {
          console.error('初始化表格数据出错：', e);
        }
      }
    },
    
    operate(index) {
      this.$Message.info(`Clicked operate button at row ${index}`);
    },
  },
  computed: {
    isFormValid() {
      // 检查必填项是否都已填写
      if (!this.selectedItem || !this.operator || !this.selectedPrinciple || !this.selectCount) {
        return false;
      }

      // 检查是否至少选择了一个选项（楼栋、层、户）
      if (!this.buildingSelected && !this.floorSelected && !this.roomSelected) {
        return false;
      }

      // 检查所有已启用的输入框是否都有值且格式正确
      for (const group of this.buildingGroups) {
        if (this.buildingSelected && !group.building) {
          return false;
        }
        if (this.floorSelected && !group.floor) {
          return false;
        }
        if (this.roomSelected && !group.room) {
          return false;
        }
      }

      // 检查是否存在错误信息
      for (const errors of Object.values(this.errorMessages)) {
        if (errors && Object.values(errors).some(error => error)) {
          return false;
        }
      }

      return true;
    }
  }
}
</script>

<style scoped>
.area-selection-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.config-panel, .result-panel, .record-panel {
  border: 1px solid #e8eaec;
  border-radius: 4px;
}

/* 添加水平布局容器 */
.area-selection-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.horizontal-layout {
  display: flex;
  gap: 20px;
}

.config-panel {
  flex: 1;  /* 固定宽度 */
}

.result-panel {
  flex: 1;  /* 占据剩余空间 */
}

.panel-header {
  background-color: #f8f8f9;
  padding: 10px 15px;
  font-weight: bold;
  border-bottom: 1px solid #e8eaec;
  display: flex;
  align-items: center;
}

.panel-header .ivu-icon {
  margin-right: 5px;
  color: #2d8cf0;
}

.panel-content {
  padding: 20px;
}

/* 高层选择区域样式 */
.high-level-item {
  margin-bottom: 0;
}

.high-level-content {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.checkbox-row {
  display: flex;
  gap: 30px;
}

.input-row {
  display: flex;
  gap: 10px;
  align-items: flex-start;
}

.input-row .ivu-form-item {
  margin-right: 0;
}

.input-row .ivu-input-wrapper {
  width: 240px;  /* 增加输入框宽度以适应更长的内容 */
}

.symbol-input {
  display: flex;
  align-items: center;
  gap: 5px;
  color: #888;
  font-size: 12px;
}

.symbol {
  display: inline-block;
  border-bottom: 1px solid #ccc;
  padding: 0 5px;
}

/* 新增类型按钮 */
.add-new-type {
  color: #2d8cf0;
  cursor: pointer;
  margin: 15px 0;
  display: flex;
  align-items: center;
}

.add-new-type .ivu-icon {
  margin-right: 5px;
}

/* 按钮区域 */
.action-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.table-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.building-group {
  margin-bottom: 20px;  /* 增加间距以容纳错误信息 */
}

.building-group:last-child {
  margin-bottom: 0;
}

/* 添加错误提示样式 */
.input-row .ivu-form-item-error-tip {
  position: absolute;
  top: 100%;
  left: 0;
  padding-top: 5px;
  line-height: 1;
  color: #ed4014;
  font-size: 12px;
}
</style>

