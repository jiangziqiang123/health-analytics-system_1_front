<template>
  <div class="page-container">
    <div class="top-bar">
      <div class="top-left">
        <Timer />
        <Role /> <!-- 新增：Timer后插入Role组件 -->
      </div>
      <div class="top-center">
        <Title />
      </div>
      <div class="top-right">
        <Settings />
        <Exit /> <!-- 新增：Settings后插入Exit组件 -->
      </div>
    </div>



    <div class="bottom-align" v-if="!mapDataStore.isLoading && !mapDataStore.error">
      <ChinaMap
          @region-clicked="handleRegionClick"
          :provinceData="mapDataStore.provinceData"
          :scatterData="mapDataStore.scatterData"
          :pieSeriesData="mapDataStore.pieSeriesData"
      />
    </div>
<!--    医疗机构卡片-->


    <div class="card-display-area">
      <card :componentsList="myCustomComponents1"></card>
    </div>
    <div class="card-display-area2">
      <card :componentsList="myCustomComponents2"></card>
    </div>
    <div class="card-display-area3">
      <card :componentsList="myCustomComponents3"></card>
    </div>
    <div class="card-display-area4">
      <card :componentsList="myCustomComponents4"></card>
    </div>
    <div class="card-display-area5">
      <card :componentsList="myCustomComponents5"></card>
    </div>
    <div class="card-display-area6">
      <card :componentsList="myCustomComponents6"></card>
    </div>
    <div class="scroll-display-area">
      <scroll></scroll>
    </div>
    <InteractiveButtonContainer
        :buttonsRow1="myCustomButtonsRow1"
        :buttonsRow2="myCustomButtonsRow2"
        @button-clicked="handleChildButtonClick"
    />
    <div v-if="mapDataStore.isLoading" class="loading-overlay tech-loading">
      <div class="loading-spinner"></div>
      <p>数据加载中，请稍候...</p>
    </div>
    <div v-if="mapDataStore.error" class="error-message tech-error">
      <p>加载数据失败：{{ mapDataStore.error }}</p>
      <el-button type="primary" size="small" @click="mapDataStore.fetchMapData()">重试</el-button>
    </div>
  </div>
</template>

<script>
import ChinaMap from '../../components/Map.vue';

import Timer from '../../components/Timer.vue';// 导入 Timer 组件
import Title from '../../components/Title.vue';
import { ref, onMounted } from 'vue';
import { useMapDataStore } from '@/stores/TotalData.js';
import {useRegionStore} from '@/stores/RegionData.js';
import {ElMessage} from "element-plus";
import card from '../card/CardContainer.vue'; // 导入 CardContainer.vue 组件
import InteractiveButtonContainer from '@/components/ButtonContainer.vue';
import scroll from './scroll.vue';

import Settings from "@/components/Settings.vue";
import Role from '../../components/Role.vue'; // 新增
import Exit from '../../components/Exit.vue'; // 新增


import MyComponentA from '../card/MyComponentA.vue';
import MyComponentB from '../card/MyComponentB.vue';
import MyComponentC from '../card/MyComponentC.vue';
import MyComponentD from '../card/MyComponentD.vue';
import ServiceforML from "@/pages/card/ServiceforML.vue";
import PopulationLine from "@/pages/card/PopulationLine.vue";
import RankforPop from "@/pages/card/RankforPop.vue";


export default {
  components: {
    Settings,
    Timer,
    Title,
    ChinaMap,
    card,
    scroll,

    Role, // 新增
    Exit,  // 新增

    InteractiveButtonContainer,

  },
  setup() {
    const myCustomComponents1 = ref([
      MyComponentA,
      MyComponentB,
      MyComponentC,
      MyComponentD,
      ServiceforML,
      PopulationLine,
      RankforPop,
    ]);


    const mapDataStore = useMapDataStore();
    const regionStore = useRegionStore();

    const handleRegionClick = (regionName) => {
      regionStore.setSelectedRegion(regionName)
      if (regionStore.selectedRegionId !== 0) {
        ElMessage({
          message: `你点击了：${regionStore.getDisplayRegion} (ID: ${regionStore.getRegionId})`,
          type: 'info',
          duration: 2000,
          customClass: 'tech-message'
        });
      } else {
        ElMessage({
          message: `取消选择：${regionName} (当前全国视图, ID: ${regionStore.getRegionId})`,
          type: 'info',
          duration: 2000,
          customClass: 'tech-message'
        });
      }
    };
    const handleChildButtonClick = (event) => {
      console.log(`按钮点击事件：行 ID ${event.id}`);

      mapDataStore.fetchsatterdata(event.id);


    };
    const myCustomButtonsRow1 = ref([
      { id: 'populationData', label: '人口' },
      { id: 'institutionData', label: '医疗机构' },
      { id: 'personnelData', label: '医疗人员' },
      { id: 'bedData', label: '床位总数' },
      { id: 'outpatientVisitsData', label: '就诊人数' },
      { id:'totalCostData',label:'医疗费用'},
    ]);

    const myCustomButtonsRow2 = ref([
      { id: 'view_map', label: '查看地图' },
      { id: 'chart_display', label: '图表展示' },
      { id: 'table_data', label: '表格数据' },
      { id: 'export_data', label: '导出数据' },
    ]);

    const myCustomComponents2 = ref([
      MyComponentB,
    ]);
    const myCustomComponents3 = ref([
      MyComponentC,
    ]);
    const myCustomComponents4 = ref([
      MyComponentD,
    ]);
    const myCustomComponents5 = ref([
      PopulationLine,
      RankforPop,
    ]);
    const myCustomComponents6 = ref([
      ServiceforML,
    ]);
    onMounted(() => {
      mapDataStore.fetchcountryData()
      if (mapDataStore.provinceData.length === 0 && !mapDataStore.isLoading) {
        mapDataStore.fetchMapData();
      }
    });

    return {
      handleRegionClick,
      handleChildButtonClick,
      mapDataStore,
      myCustomButtonsRow1,
      myCustomButtonsRow2,
      myCustomComponents1: myCustomComponents1,
      myCustomComponents2: myCustomComponents2,
      myCustomComponents3: myCustomComponents3,
      myCustomComponents4: myCustomComponents4,
      myCustomComponents5: myCustomComponents5,
      myCustomComponents6: myCustomComponents6,
    };
  },
};
</script>
<style>
body {
  margin: 0;
  padding: 0;
  font-family: 'Inter', sans-serif, 'Microsoft YaHei', 'PingFang SC';
  background: linear-gradient(135deg, #0a192f 0%, #000a1a 100%);
  color: #e0f2f7;
  min-height: 100vh;
  overflow-x: hidden;
}
.page-container {


  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;

  position: relative;
}

/* 顶部区域固定高度 */
.top-bar {
  position: relative;
  width: 100%;
  height: 80px; /* 根据 Timer 和 Title 高度调整 */
}


/* 左上角 */
.top-left {
  position: absolute;
  top: 10px;
  left: 20px;
  z-index: 10;

  /* card 组件的包裹 div 样式，定位在左上方 */

  .card-display-area {
    position: fixed;
    top: 7%; /* 与地图顶部对齐 */
    left: 1%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .card-display-area2 {
    position: fixed;
    top: 30%; /* 与地图顶部对齐 */
    left: 1%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .card-display-area3 {
    position: fixed;
    top: 53%; /* 与地图顶部对齐 */
    left: 1%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .Dataget-display-area {
    position: fixed;
    top: 70%; /* 与地图顶部对齐 */
    left: 1%; /* 距离左侧边缘 */
    width: 30%; /* 限定宽度 */
    height: 30%; /* 限定高度，与地图高度相同，保持视觉平衡 */

  }

  .card-display-area4 {
    position: fixed;
    top: 7%; /* 与地图顶部对齐 */
    left: 77%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .card-display-area5 {
    position: fixed;
    top: 30%; /* 与地图顶部对齐 */
    left: 77%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .card-display-area6 {
    position: fixed;
    top: 53%; /* 与地图顶部对齐 */
    left: 77%; /* 距离左侧边缘 */
    width: 20%; /* 限定宽度 */
    height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
    box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
    overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
    background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

    /* 边框效果：通过多层box-shadow模拟发光边框 */
    border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
    box-shadow: 0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */ 0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */ inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

    /* 圆角：如果图片中有圆角，可以添加 */
    border-radius: 8px; /* 适当的圆角 */
  }

  .scroll-display-area {
    position: absolute;
    left: 31%;
    top: 70%;
    width: 40%;

  }

  /* 中间 */

  .top-center {
    position: absolute;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
  }

  /* 右上角 */

  .top-right {
    position: absolute;
    top: 10px;
    right: 20px;
    z-index: 10;
  }
}



</style>

<style>
/* 全局科技风主题 - 保持不变，它影响 body */
/* 全局科技风主题 - 保持不变，它影响 body */
body {
  margin: 0;
  padding: 0;
  font-family: 'Inter', sans-serif, 'Microsoft YaHei', 'PingFang SC';
  background: linear-gradient(135deg, #0a192f 0%, #000a1a 100%);
  color: #e0f2f7;
  min-height: 100vh;
  overflow-x: hidden;
}

/* 页面容器基础样式 - 保持不变 */
.page-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}

/* ChinaMap 容器 - 严格保持原有位置和大小 */
.bottom-align {
  position: fixed;
  top: 7%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  justify-content: center;
  width: 38%;
  height: 52%;
  padding: 1px;
  border-radius: 1px;
  border: 2px solid transparent;
  background-origin: border-box;
  background-clip: padding-box, border-box;
  overflow: hidden;
}

/* card 组件的包裹 div 样式，定位在左上方 */
.card-display-area {
  position: fixed;
  top: 7%; /* 与地图顶部对齐 */
  left: 1%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.card-display-area2 {
  position: fixed;
  top: 30%; /* 与地图顶部对齐 */
  left: 1%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.card-display-area3 {
  position: fixed;
  top: 53%; /* 与地图顶部对齐 */
  left: 1%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.Dataget-display-area{
  position: fixed;
  top: 70%; /* 与地图顶部对齐 */
  left: 1%; /* 距离左侧边缘 */
  width: 30%; /* 限定宽度 */
  height: 30%; /* 限定高度，与地图高度相同，保持视觉平衡 */

}
.card-display-area4 {
  position: fixed;
  top: 7%; /* 与地图顶部对齐 */
  left: 77%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.card-display-area5 {
  position: fixed;
  top: 30%; /* 与地图顶部对齐 */
  left: 77%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.card-display-area6 {
  position: fixed;
  top: 53%; /* 与地图顶部对齐 */
  left: 77%; /* 距离左侧边缘 */
  width: 20%; /* 限定宽度 */
  height: 20%; /* 限定高度，与地图高度相同，保持视觉平衡 */
  box-sizing: border-box; /* 确保 padding 不增加元素总尺寸 */
  overflow: hidden; /* 确保内部内容不会超出带有圆角的边框 */
  background: linear-gradient(135deg, rgba(3, 4, 94, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);

  /* 边框效果：通过多层box-shadow模拟发光边框 */
  border: 3px solid rgba(74, 207, 255, 0.2); /* 内部更细的弱透明边框 */
  box-shadow:
      0 0 10px rgba(74, 207, 255, 0.4), /* 内部蓝色微光 */
      0 0 20px rgba(74, 207, 255, 0.2), /* 外部蓝色光晕 */
      inset 0 0 10px rgba(74, 207, 255, 0.3); /* 内部向内发光，增加立体感 */

  /* 圆角：如果图片中有圆角，可以添加 */
  border-radius: 8px; /* 适当的圆角 */
}
.scroll-display-area {
  position: absolute;
  left: 31%;
  top: 70%;
  width: 40%;
}


/* 加载和错误状态显示 - 保持原有定位和尺寸，仅优化视觉效果 */
.loading-overlay, .error-message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: #fff;
  padding: 20px 30px;
  border-radius: 10px;
  font-size: 1.2em;
  z-index: 1000;
  text-align: center;
}

.error-message {
  background-color: rgba(255, 0, 0, 0.7);
}

/* selected-region-display 样式 */
.selected-region-display {
  margin-top: 550px;
  padding: 10px 20px;
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(0, 150, 255, 0.5);
  border-radius: 5px;
  color: #00e0ff;
  font-size: 1.1em;
  font-weight: 500;
  text-shadow: 0 0 5px rgba(0, 180, 255, 0.5);
  align-self: center;
  z-index: 5;
}
.page-header{
  height: 7%;
}

/* 加载和错误覆盖层 - 科技风视觉优化 */
.loading-overlay.tech-loading, .error-message.tech-error {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.9);
  color: #00e0ff;
  font-size: 1.5em;
  z-index: 2000;
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
}

.loading-spinner {
  border: 6px solid rgba(0, 150, 255, 0.3);
  border-top: 6px solid #00e0ff;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin-bottom: 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-message.tech-error {
  background-color: rgba(139, 0, 0, 0.85);
  border: 1px solid rgba(255, 0, 0, 0.7);
  box-shadow: 0 0 15px rgba(255, 0, 0, 0.5);
  color: #ffcccc;
}

.error-message.tech-error .el-button {
  margin-top: 15px;
  background-color: #007bff;
  border-color: #007bff;
  color: white;
  transition: background-color 0.3s ease;
}

.error-message.tech-error .el-button:hover {
  background-color: #0056b3;
  border-color: #0056b3;
}

/* Element Plus Message Box 科技风样式 - 如果 main.vue 中也会用到 ElMessage，则这些样式也需要 */
/* 这里我把它们保留在 main.vue 的 style 中，因为 handleRegionClick 还在使用 ElMessage */
.el-message.tech-message {
  background-color: rgba(26, 42, 58, 0.9);
  border: 1px solid rgba(0, 150, 255, 0.5);
  box-shadow: 0 0 10px rgba(0, 150, 255, 0.3);
  color: #e0f2f7;
}
.el-message.tech-message .el-message__content {
  color: inherit;
}
.el-message.tech-message.el-message--info {
  background-color: rgba(26, 42, 58, 0.9);
}
.el-message.tech-message.el-message--success {
  background-color: rgba(26, 42, 58, 0.9);
}


.top-left {
  position: absolute;
  top: 10px;
  left: 20px;
  z-index: 10;
}

/* 中间 */
.top-center {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}

/* 右上角 */
.top-right {
  position: absolute;
  top: 10px;
  right: 20px;
  z-index: 10;
}
</style>
