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
            <FormItem label="抽选阶段" required>
              <Select v-model="selectedStage" placeholder="请选择抽选阶段">
                <Option value="主体" label="主体"></Option>
                <Option value="砌筑" label="砌筑"></Option>
                <Option value="抹灰" label="抹灰"></Option>
                <Option value="门窗" label="门窗"></Option>
                <Option value="精装修" label="精装修"></Option>
                <Option value="钢结构" label="钢结构"></Option>
              </Select>
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
                        style="width: 150px;"
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
                        style="width: 150px;"
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
                        style="width: 150px;"
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
                  <span>栋号、楼层和户号均支持：单个数字(1)，范围(3-4)，多个范围(3-4,5-8,10-12)</span>
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
          <Table 
            border 
            :columns="columns" 
            :data="tableData" 
            empty-text="暂无数据，请在左侧'配置抽测范围'"
            @on-selection-change="handleSelectionChange"
          >
          </Table>
          <div class="table-actions">
            <Button type="default" @click="handleClear">清空列表</Button>
            <Button 
              type="primary" 
              @click="handleBatchSelect"
              :disabled="selectedRows.length === 0"
            >统一抽取</Button>
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
      selectedItem: '',
      selectedPrinciple: '随机抽取',
      selectCount: '',
      operator: '',
      selectedStage: '',
      
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

      // 添加选中数据数组
      selectedRows: [],
      
      // 抽测范围列表
      columns: [
        { 
          type: 'selection', 
          width: 60, 
          align: 'center'
        },
        { title: '抽取项目', key: 'item' },
        { 
          title: '抽取范围', 
          key: 'range',
          render: (h, params) => {
            // 处理范围显示，将"-"替换为空格，使栋、层、户之间更易区分
            const range = params.row.range;
            // 替换"栋-"为"栋 "，"层-"为"层 "，但保留数字之间的"-"
            const formattedRange = range.replace(/栋-/g, '栋 ').replace(/层-/g, '层 ');
            return h('span', formattedRange);
          }
        },
        { title: '抽取原则', key: 'principle', width: 95 },
        { title: '抽测数量', key: 'count', width: 95 },
        { title: '抽选阶段', key: 'stage', width: 95 },
        { title: '操作', key: 'action', width: 200, render: (h, params) => {
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
                  this.handleSelect(params.row);
                }
              }
            }, '抽取'),
            h('Button', {
              props: {
                type: 'info',
                size: 'small'
              },
              style: {
                marginRight: '5px'
              },
              on: {
                click: () => {
                  this.handleEdit(params.row, params.index);
                }
              }
            }, '修改'),
            h('Button', {
              props: {
                type: 'error',
                size: 'small'
              },
              on: {
                click: () => {
                  this.handleDelete(params.index);
                }
              }
            }, '删除')
          ]);
        }}
      ],
      tableData: [],
      
      // 抽取记录
      recordColumns: [
        { title: '记录组', key: 'group', width: 100 },
        { title: '抽取项目', key: 'item' },
        { title: '抽选阶段', key: 'stage' },
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
        { title: '操作', key: 'action', width: 280, render: (h, params) => {
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
                  this.showRecordResult(params.row);
                }
              }
            }, '结果展示'),
            // h('Button', {
            //   props: {
            //     size: 'small'
            //   },
            //   style: {
            //     marginRight: '5px'
            //   }
            // }, '复制'),
            h('Button', {
              props: {
                type: 'warning',
                size: 'small',
                disabled: params.row.status === '作废'
              },
              style: {
                marginRight: '5px'
              },
              on: {
                click: () => {
                  this.handleInvalidate(params.index);
                }
              }
            }, '作废'),
            h('Button', {
              props: {
                type: 'error',
                size: 'small'
              },
              on: {
                click: () => {
                  this.handleDeleteRecord(params.index);
                }
              }
            }, '删除')
          ])
        }}
      ],
      recordData: [],

      // 新增: 错误消息存储
      errorMessages: [],

      // 添加抽取历史权重记录
      selectionHistory: {},

      // 添加编辑状态数据
      editingIndex: -1, // -1 表示不在编辑状态

      // 新增: 作废原因存储
      invalidateReason: '',
    }
  },
  created() {
    // 初始化错误消息数组
    this.initErrorMessages();
    // 初始化表格数据
    this.initTableData();
    // 初始化记录数据
    this.initRecordData();
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
      
      // 对所有类型使用相同的验证逻辑
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
    
    // 修改handleSubmit方法
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
          operator: this.operator,
          stage: this.selectedStage
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

        // 如果是编辑状态，替换原数据
        if (this.editingIndex !== -1) {
          dataList[this.editingIndex] = newItem;
          this.$Message.success('修改成功');
        } else {
          // 添加新数据
          dataList.push(newItem);
          this.$Message.success('保存成功');
        }

        // 保存到本地存储
        localStorage.setItem('areaSelectionData', JSON.stringify(dataList));

        // 更新表格数据
        this.tableData = dataList;

        // 重置编辑状态
        this.editingIndex = -1;

        // 清空表单数据
        this.resetForm();
      } else {
        this.$Message.error('请修正表单中的错误');
      }
    },
    
    // 修改resetForm方法
    resetForm() {
      this.selectedItem = '';
      this.operator = '';
      this.selectCount = '2';
      this.selectedStage = '';
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
      // 重置编辑状态
      this.editingIndex = -1;
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
    
    // 解析带范围的字符串
    extractRangeString(rangeStr) {
      console.log('解析范围字符串:', rangeStr);
      
      // 分析范围字符串的结构，如"3-4栋-5-8层-12-16户"
      const result = {
        building: '',
        floor: '',
        room: ''
      };
      
      try {
        // 提取栋号部分，使用非贪婪匹配确保只匹配到栋号
        const buildingMatch = rangeStr.match(/(.+?)栋/);
        if (buildingMatch) {
          result.building = buildingMatch[1];
          console.log('提取到栋号部分:', result.building);
        }
        
        // 提取层号部分，匹配栋和层之间的内容
        const floorMatch = rangeStr.match(/栋-(.+?)层/);
        if (floorMatch) {
          result.floor = floorMatch[1];
          console.log('提取到层号部分:', result.floor);
        }
        
        // 提取户号部分，匹配层和户之间的内容
        const roomMatch = rangeStr.match(/层-(.+?)户/);
        if (roomMatch) {
          result.room = roomMatch[1];
          console.log('提取到户号部分:', result.room);
        }
      } catch (error) {
        console.error('解析范围字符串出错:', error);
      }
      
      return result;
    },
    
    // 解析单个范围字符串，返回所有可能的组合
    parseRangeString(rangeStr) {
      console.log('进入parseRangeString，处理范围:', rangeStr);
      const combinations = [];
      
      // 首先提取栋号
      let building = '';
      let floorStr = '';
      let roomStr = '';
      
      // 处理栋号
      const buildingMatch = rangeStr.match(/(\d+)栋/);
      if (buildingMatch) {
        building = buildingMatch[1];
        console.log('找到栋号:', building);
      }
      
      // 处理层号
      const floorMatch = rangeStr.match(/(\d+(?:-\d+)?)层/);
      if (floorMatch) {
        floorStr = floorMatch[1];
        console.log('找到层号表达式:', floorStr);
      }
      
      // 处理户号
      const roomMatch = rangeStr.match(/(\d+(?:-\d+)?)户/);
      if (roomMatch) {
        roomStr = roomMatch[1];
        console.log('找到户号表达式:', roomStr);
      }
      
      // 解析层号范围
      const floors = this.parseNumberRanges(floorStr);
      console.log('解析后的层数:', floors);
      
      // 解析户号范围
      const rooms = this.parseNumberRanges(roomStr);
      console.log('解析后的户号:', rooms);
      
      // 根据选择的项目生成组合
      if (building) {
        if (floors.length === 0 && rooms.length === 0) {
          // 只有栋号
          combinations.push(`${building}栋`);
        } else if (floors.length > 0 && rooms.length === 0) {
          // 有栋号和层号
          floors.forEach(floor => {
            combinations.push(`${building}栋${floor}层`);
          });
        } else if (floors.length > 0 && rooms.length > 0) {
          // 有栋号、层号和户号
          floors.forEach(floor => {
            rooms.forEach(room => {
              combinations.push(`${building}栋${floor}层${room}户`);
            });
          });
        }
      }
      
      console.log('生成的组合结果:', combinations);
      return combinations;
    },
    
    // 解析数字范围，如 "2-7,8-16" 返回 [2,3,4,5,6,7,8,9,10,11,12,13,14,15,16]
    parseNumberRanges(rangeStr) {
      console.log('解析数字范围:', rangeStr);
      if (!rangeStr || rangeStr.trim() === '') {
        return [];
      }
      
      const result = [];
      
      try {
        // 标准化输入，处理中文字符
        rangeStr = rangeStr.replace(/，/g, ',').replace(/[~～]/g, '-');
        console.log('标准化后的范围字符串:', rangeStr);
        
        // 分割逗号，处理多个范围
        const ranges = rangeStr.split(',');
        console.log('分割后的范围列表:', ranges);
        
        ranges.forEach(range => {
          range = range.trim();
          console.log('处理范围:', range);
          
          if (range.includes('-')) {
            // 处理范围，如 "2-7"
            const [start, end] = range.split('-').map(num => parseInt(num, 10));
            console.log(`范围 ${start} 到 ${end}`);
            
            if (!isNaN(start) && !isNaN(end) && start <= end) {
              for (let i = start; i <= end; i++) {
                result.push(i);
                console.log(`添加数字: ${i}`);
              }
            } else {
              console.warn(`无效范围: ${range}, start=${start}, end=${end}`);
            }
          } else if (/^\d+$/.test(range)) {
            // 处理单个数字
            const num = parseInt(range, 10);
            if (!isNaN(num)) {
              result.push(num);
              console.log(`添加单个数字: ${num}`);
            } else {
              console.warn(`无效数字: ${range}`);
            }
          } else {
            console.warn(`无法解析的格式: ${range}`);
          }
        });
      } catch (error) {
        console.error('解析数字范围时出错:', error);
      }
      
      console.log('最终解析结果:', result);
      return result;
    },
    
    // 解析完整的范围数据
    parseFullRange(rangeStr) {
      console.log('解析完整范围:', rangeStr);
      
      // 检查是否有多个范围（以中文逗号分隔）
      const ranges = rangeStr.split('，');
      console.log('分割后的范围数量:', ranges.length);
      
      const allCombinations = [];
      
      // 处理每一个范围表达式
      for (const range of ranges) {
        console.log(`处理范围表达式: ${range}`);
        
        try {
          // 使用新的正则表达式提取各部分
          let buildingPart = '';
          let floorPart = '';
          let roomPart = '';
          
          // 提取栋号部分，使用非贪婪匹配确保只匹配到栋号
          const buildingMatch = range.match(/(.+?)栋/);
          if (buildingMatch) {
            buildingPart = buildingMatch[1];
            console.log('提取到栋号部分:', buildingPart);
          }
          
          // 提取层号部分，匹配栋和层之间的内容
          const floorMatch = range.match(/栋-(.+?)层/);
          if (floorMatch) {
            floorPart = floorMatch[1];
            console.log('提取到层号部分:', floorPart);
          }
          
          // 提取户号部分，匹配层和户之间的内容
          const roomMatch = range.match(/层-(.+?)户/);
          if (roomMatch) {
            roomPart = roomMatch[1];
            console.log('提取到户号部分:', roomPart);
          }
          
          // 解析各个范围
          const buildings = this.parseNumberRanges(buildingPart);
          const floors = this.parseNumberRanges(floorPart);
          const rooms = this.parseNumberRanges(roomPart);
          
          console.log('解析后的栋号列表:', buildings);
          console.log('解析后的层号列表:', floors);
          console.log('解析后的户号列表:', rooms);
          
          // 生成所有可能的组合
          buildings.forEach(building => {
            if (floors.length === 0 && rooms.length === 0) {
              // 只有栋号
              allCombinations.push(`${building}栋`);
            } else if (floors.length > 0 && rooms.length === 0) {
              // 有栋号和层号
              floors.forEach(floor => {
                allCombinations.push(`${building}栋${floor}层`);
              });
            } else if (floors.length > 0 && rooms.length > 0) {
              // 有栋号、层号和户号
              floors.forEach(floor => {
                rooms.forEach(room => {
                  allCombinations.push(`${building}栋${floor}层${room}户`);
                });
              });
            }
          });
        } catch (error) {
          console.error('解析范围时出错:', error);
        }
      }
      
      console.log('所有可能组合:', allCombinations);
      return allCombinations;
    },
    
    // 重置权重
    resetWeights(pool) {
      const weights = {};
      pool.forEach(item => {
        weights[item] = 1;
      });
      return weights;
    },

    // 更新权重
    updateWeights(weights, selected) {
      // 被选中的项权重降低
      selected.forEach(item => {
        if (weights[item]) {
          weights[item] = Math.max(0.1, weights[item] - 0.3);
        }
      });

      // 未被选中的项权重提高
      Object.keys(weights).forEach(item => {
        if (!selected.includes(item)) {
          weights[item] = Math.min(2, weights[item] + 0.1);
        }
      });

      return weights;
    },

    // 随机抽取指定数量的项目
    randomSelect(array, count) {
      console.log('执行随机抽取, 项目池:', array, '需要抽取数量:', count);
      
      if (!array || array.length === 0) {
        console.log('项目池为空，返回空数组');
        return [];
      }

      // 对于测试，先简单处理，直接随机抽取
      if (array.length <= count) {
        console.log('项目池数量小于等于需要抽取的数量，返回所有项目');
        return [...array];
      }
      
      // 随机打乱数组并取前count个
      const shuffled = [...array].sort(() => Math.random() - 0.5);
      const result = shuffled.slice(0, count);
      console.log('抽取结果:', result);
      return result;
    },

    // 清空历史记录
    clearHistory() {
      this.selectionHistory = {};
      this.$Message.success('已重置抽取权重');
    },
    
    // 修改handleSelect方法
    handleSelect(row) {
      console.log('----------------------------------------');
      console.log('开始抽取, 范围:', row.range);
      console.log('抽取数量:', row.count);
      
      // 解析范围数据
      const pool = this.parseFullRange(row.range);
      console.log('解析后的可抽取池:', pool);
      console.log('可抽取总数:', pool.length);
      
      // 获取需要抽取的数量
      const count = parseInt(row.count);
      
      if (pool.length < count) {
        this.$Message.warning(`可抽取的总数(${pool.length})小于需要抽取的数量(${count})`);
        return;
      }
      
      if (pool.length === 0) {
        this.$Message.warning('可抽取的范围为空，请检查配置');
        return;
      }
      
      // 随机抽取
      const selected = this.randomSelect(pool, count);
      console.log('抽取结果:', selected);
      
      // 格式化范围显示
      const formattedRange = row.range.replace(/栋-/g, '栋 ').replace(/层-/g, '层 ');
      
      // 显示抽取结果
      this.$Modal.info({
        title: '抽取结果',
        content: `
          <div style="margin-bottom: 15px;">
            <div><strong>抽取项目：</strong>${row.item}</div>
            <div><strong>抽选阶段：</strong>${row.stage}</div>
            <div><strong>抽取数量：</strong>${selected.length}</div>
            <div><strong>抽取范围：</strong>${formattedRange}</div>
            <div><strong>抽取原则：</strong>${row.principle}</div>
          </div>
          <div style="margin-bottom: 10px;"><strong>抽取结果：</strong></div>
          <div style="max-height: 300px; overflow-y: auto; padding: 10px; background-color: #f8f8f9; border-radius: 4px;">
            ${selected.map((item, index) => `
              <div style="margin-bottom: 5px;">${index + 1}. ${item}</div>
            `).join('')}
          </div>
        `,
        width: 500
      });
      
      // 添加到抽取记录
      const record = {
        group: `第${this.recordData.length + 1}组`,
        item: row.item,
        stage: row.stage,
        status: '正常',
        description: '',
        operator: row.operator,
        time: new Date().toLocaleString(),
        result: selected
      };
      
      this.recordData.push(record);
      
      // 保存记录到本地存储
      this.saveRecords();
    },
    
    // 保存记录到本地存储
    saveRecords() {
      localStorage.setItem('areaSelectionRecords', JSON.stringify(this.recordData));
    },
    
    // 初始化记录数据
    initRecordData() {
      const savedRecords = localStorage.getItem('areaSelectionRecords');
      if (savedRecords) {
        try {
          this.recordData = JSON.parse(savedRecords);
        } catch (e) {
          console.error('解析记录数据出错：', e);
          this.recordData = [];
        }
      }
    },

    // 修改handleEdit方法
    handleEdit(row, index) {
      console.log('开始编辑数据:', row);
      console.log('需要解析的范围:', row.range);
      
      // 设置编辑状态
      this.editingIndex = index;

      // 将数据填充到表单中
      this.selectedItem = row.item;
      this.selectedPrinciple = row.principle;
      this.selectCount = row.count;
      this.operator = row.operator;
      this.selectedStage = row.stage;

      // 解析范围数据
      const ranges = row.range.split('，');
      console.log('分割后的范围:', ranges);
      
      this.buildingGroups = [];
      
      // 为每个范围创建一个组
      ranges.forEach((range, idx) => {
        console.log(`处理第${idx+1}个范围: ${range}`);
        
        // 使用专门的函数提取范围数据
        const rangeData = this.extractRangeString(range);
        console.log(`第${idx+1}个范围提取结果:`, rangeData);
        
        this.buildingGroups.push({
          building: rangeData.building,
          floor: rangeData.floor,
          room: rangeData.room
        });
      });

      console.log('设置的表单数据:', this.buildingGroups);
      
      // 设置选中状态
      this.buildingSelected = true;
      this.floorSelected = ranges.some(r => r.includes('层'));
      this.roomSelected = ranges.some(r => r.includes('户'));

      // 提示用户
      this.$Message.info('请在左侧表单中修改数据后保存');
    },

    // 处理删除操作
    handleDelete(index) {
      this.$Modal.confirm({
        title: '确认删除',
        content: '确定要删除这条记录吗？',
        onOk: () => {
          this.tableData.splice(index, 1);
          localStorage.setItem('areaSelectionData', JSON.stringify(this.tableData));
          this.$Message.success('删除成功');
        }
      });
    },

    // 批量抽取方法的修改
    handleBatchSelect() {
      if (this.selectedRows.length === 0) {
        this.$Message.warning('请先选择要抽取的项目');
        return;
      }

      // 存储所有抽取结果
      const allResults = [];
      let hasError = false;

      // 对每个选中的项目进行抽取
      this.selectedRows.forEach(row => {
        // 解析范围数据
        const pool = this.parseFullRange(row.range);
        const count = parseInt(row.count);

        if (pool.length < count) {
          this.$Message.warning(`${row.item}可抽取的总数(${pool.length})小于需要抽取的数量(${count})`);
          hasError = true;
          return;
        }

        // 随机抽取
        const selected = this.randomSelect(pool, count);

        // 添加到结果集
        allResults.push({
          item: row.item,
          stage: row.stage,
          count: selected.length,
          results: selected,
          operator: row.operator
        });

        // 添加到抽取记录
        const record = {
          group: `第${this.recordData.length + 1}组`,
          item: row.item,
          stage: row.stage,
          status: '正常',
          description: '',
          operator: row.operator,
          time: new Date().toLocaleString(),
          result: selected
        };
        
        this.recordData.push(record);
      });

      // 如果有错误则不显示结果
      if (hasError) {
        return;
      }

      // 显示批量抽取结果
      this.$Modal.info({
        title: '批量抽取结果',
        content: `
          <div style="max-height: 500px; overflow-y: auto;">
            ${allResults.map(result => {
              // 查找原始行数据以获取范围
              const originalRow = this.selectedRows.find(row => row.item === result.item);
              const formattedRange = originalRow ? originalRow.range.replace(/栋-/g, '栋 ').replace(/层-/g, '层 ') : '';
              
              return `
                <div style="margin-bottom: 20px;">
                  <div style="font-weight: bold; margin-bottom: 10px;">
                    ${result.item}（抽取数量：${result.count}）
                  </div>
                  <div style="margin-bottom: 5px;">
                    <span style="color: #666;">抽取范围：${formattedRange}</span>
                  </div>
                  <div style="margin-left: 20px;">
                    ${result.results.map((item, index) => `${index + 1}. ${item}`).join('<br>')}
                  </div>
                </div>
              `;
            }).join('<hr style="margin: 15px 0;">')}
          </div>
        `,
        width: 500
      });

      // 保存记录到本地存储
      this.saveRecords();

      // 清除选中状态
      this.selectedRows = [];
    },

    // 表格选择事件处理
    handleSelectionChange(selection) {
      this.selectedRows = selection;
    },

    // 显示记录结果
    showRecordResult(record) {
      if (!record.result || record.result.length === 0) {
        this.$Message.warning('没有找到抽取结果');
        return;
      }

      this.$Modal.info({
        title: '抽取结果',
        content: `
          <div style="margin-bottom: 15px;">
            <div><strong>记录组：</strong>${record.group}</div>
            <div><strong>抽取项目：</strong>${record.item}</div>
            <div><strong>抽选阶段：</strong>${record.stage || '未指定'}</div>
            <div><strong>抽取人：</strong>${record.operator}</div>
            <div><strong>抽取时间：</strong>${record.time}</div>
            <div><strong>抽取数量：</strong>${record.result.length}</div>
          </div>
          <div style="margin-bottom: 10px;"><strong>抽取结果：</strong></div>
          <div style="max-height: 300px; overflow-y: auto; padding: 10px; background-color: #f8f8f9; border-radius: 4px;">
            ${record.result.map((item, index) => {
              // 格式化结果项，使栋、层、户之间更易区分
              const formattedItem = item.replace(/栋/g, '栋 ').replace(/层/g, '层 ');
              return `<div style="margin-bottom: 5px;">${index + 1}. ${formattedItem}</div>`;
            }).join('')}
          </div>
        `,
        width: 500,
        styles: {
          top: '50px'
        }
      });
    },

    // 清空列表和历史记录
    handleClear() {
      this.$Modal.confirm({
        title: '确认清空',
        content: '是否确认清空列表和抽取历史？这将重置所有抽取权重。',
        onOk: () => {
          this.tableData = [];
          this.clearHistory();
          localStorage.removeItem('areaSelectionData');
          this.$Message.success('已清空列表和抽取历史');
        }
      });
    },

    // 处理记录删除
    handleDeleteRecord(index) {
      this.$Modal.confirm({
        title: '确认删除',
        content: '确定要删除这条抽取记录吗？删除后无法恢复。',
        onOk: () => {
          this.recordData.splice(index, 1);
          this.saveRecords();
          this.$Message.success('删除成功');
        }
      });
    },

    // 处理作废操作
    handleInvalidate(index) {
      this.$Modal.confirm({
        title: '记录作废',
        render: (h) => {
          return h('div', [
            h('p', '请输入作废说明：'),
            h('Input', {
              props: {
                type: 'textarea',
                rows: 4,
                placeholder: '请输入作废原因...'
              },
              on: {
                input: (val) => {
                  this.invalidateReason = val;
                }
              }
            })
          ]);
        },
        onOk: () => {
          if (!this.invalidateReason) {
            this.$Message.warning('请输入作废说明');
            return;
          }

          // 更新记录状态
          this.recordData[index].status = '作废';
          this.recordData[index].description = this.invalidateReason;
          
          // 保存更新
          this.saveRecords();
          
          // 清空作废原因
          this.invalidateReason = '';
          
          this.$Message.success('记录已作废');
        },
        onCancel: () => {
          this.invalidateReason = '';
        }
      });
    },
  },
  computed: {
    isFormValid() {
      // 检查必填项是否都已填写
      if (!this.selectedItem || !this.operator || !this.selectedPrinciple || !this.selectCount || !this.selectedStage) {
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
  min-width: 1200px;
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
  width: 40%;  /* 固定宽度 */
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


