<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">小红书爆款图文与种草案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验不同场景的爆款标题吸睛、种草正文排版与流量话题标签，点击“一键同款生成”即可即刻核算</p>
      </div>
      <div class="showcase-badge">爆款高转化 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索种草产品、主题、受众、流派或关键字..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">种草主题：</span>“{{ sample.destination }}，要求：{{ sample.topic }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">爆款标题：</span>{{ sample.core }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic, sample.destination)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的小红书图文案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string, destination: string): void;
}>();

const categories = ['全部', '美妆护肤', '美食烘焙', '旅游避坑', '数码科技', '穿搭生活'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface XiaohongshuSample {
  id: number;
  category: string;
  destination: string;
  topic: string;
  style: string;
  core: string;
}

// 精选 30 条小红书爆款案例
const raw30Samples: XiaohongshuSample[] = [
  {
    id: 1,
    category: '美妆护肤',
    destination: '玻色因抗初老精华液',
    topic: '主打熬夜修护与去黄提亮，针对20-30岁熬夜社畜打工女生。',
    style: '爆款种草测评',
    core: '爆款标题：别再瞎交去黄税了！熬夜黄气全靠它一招翻盘，用完全脸都在发光！'
  },
  {
    id: 2,
    category: '美食烘焙',
    destination: '免烤箱空气炸锅巴斯克蛋糕',
    topic: '零失败快手甜品，适合租房党与新手，突出步骤简单与流心质感。',
    style: '干货硬核整理',
    core: '爆款标题：有手就会！空气炸锅版流心巴斯克，搅一搅就能出锅，秒杀甜品店！'
  },
  {
    id: 3,
    category: '旅游避坑',
    destination: '成都3天2晚隐秘游与美食避坑攻略',
    topic: '整理本地人常吃的苍蝇馆子，避开排队2小时的网红雷区。',
    style: '情绪共鸣避坑',
    core: '爆款标题：听劝！去成都千万别踩这6个大雷！本地土著熬夜整理的真避坑攻略！'
  },
  {
    id: 4,
    category: '数码科技',
    destination: '深色玻璃拟态 UI 设计干货与工具箱',
    topic: '适合前端开发与设计师提升审美，整理CSS透明度与阴影公式。',
    style: '干货硬核整理',
    core: '爆款标题：UI设计师必看！3分钟搞定高级深色玻璃拟态，附全套CSS参数！'
  },
  {
    id: 5,
    category: '穿搭生活',
    destination: '小个子早春胶囊衣橱10套穿搭模板',
    topic: '突出显高显瘦比例，用基本款搭出清爽韩系高级感。',
    style: '极简图文金句',
    core: '爆款标题：158小个子穿出168气场！10套早春胶囊衣橱，照着穿美翻全场！'
  },
  {
    id: 6,
    category: '美妆护肤',
    destination: '敏肌刷酸防爆痘实操全指南',
    topic: '详细拆解建立耐受、水杨酸与修护面霜的搭配公式。',
    style: '干货硬核整理',
    core: '爆款标题：保姆级刷酸全攻略！新手如何安全去闭口黑头，看这一篇就够了！'
  },
  {
    id: 7,
    category: '美食烘焙',
    destination: '打工人7天不重样减脂快手便当',
    topic: '高蛋白低卡高纤维，20分钟搞定一周工作日晚餐。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：一周减脂便当不重样！好吃不上火，掉秤快到停不下来！'
  },
  {
    id: 8,
    category: '旅游避坑',
    destination: '三亚避坑攻略与海鲜大排档真反套路',
    topic: '曝光出租车司机拉客反水套路，推荐合规平价海鲜市场。',
    style: '情绪共鸣避坑',
    core: '爆款标题：去三亚前先看这篇！出租车司机绝不会告诉你的平价海鲜避坑秘籍！'
  },
  {
    id: 9,
    category: '数码科技',
    destination: '轻薄笔记本电脑选购指南与参数对比',
    topic: '针对大学生与办公族，对比续航、屏幕质感与性价比。',
    style: '干货硬核整理',
    core: '爆款标题：大学新手怎么选笔记本？2026年爆款轻薄本避坑手册，拒绝智商税！'
  },
  {
    id: 10,
    category: '穿搭生活',
    destination: '法式慵懒风大衣与同色系穿搭技巧',
    topic: '氛围感拉满，突出羊绒质感与优雅内搭。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：越简约越高级！法式慵懒风穿搭秘诀，把平价大衣穿出大牌质感！'
  },
  {
    id: 11,
    category: '美妆护肤',
    destination: '伪素颜无瑕底妆与卡粉救星水光霜',
    topic: '解决秋冬季干皮卡粉、浮粉与斑驳，打造妈生好皮。',
    style: '爆款种草测评',
    core: '爆款标题：原相机实测！干皮卡粉星人救星，告别斑驳打造纯欲水光伪素颜！'
  },
  {
    id: 12,
    category: '美食烘焙',
    destination: '自制低糖生椰拿铁与冷萃咖啡盒',
    topic: '成本仅3元，还原知名咖啡馆风味。',
    style: '极简图文金句',
    core: '爆款标题：实现生椰拿铁自由！自制低糖冷萃复刻，味道居然比店里还香！'
  },
  {
    id: 13,
    category: '旅游避坑',
    destination: '云南大理古城与洱海骑行秘境线路',
    topic: '避开商业化喧嚣小镇，找到最美的免费看日落景点。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：大理别只去古城挤了！这几个隐秘落日视角，绝美到哭！'
  },
  {
    id: 14,
    category: '数码科技',
    destination: '降噪蓝牙耳机真实深度测评',
    topic: '评测降噪深度、音质表现与佩戴舒适度。',
    style: '爆款种草测评',
    core: '爆款标题：百元降噪耳机神仙打架！实测5款热销耳机，这才是平替天花板！'
  },
  {
    id: 15,
    category: '穿搭生活',
    destination: '复古港风卷发与精致配饰点睛法',
    topic: '突出港风明艳度，搭配耳环与发箍打造电影质感。',
    style: '极简图文金句',
    core: '爆款标题：秒变电影女主角！5分钟搞定复古港风卷发，氛围感直接拉满！'
  },
  {
    id: 16,
    category: '美妆护肤',
    destination: '极简护肤法与修护屏障水乳',
    topic: '精简护肤流程，减少肌肤负担，适合敏弱肌。',
    style: '干货硬核整理',
    core: '爆款标题：停止过度护肤！精简护肤3步法，烂脸敏肌重获新生好皮肤！'
  },
  {
    id: 17,
    category: '美食烘焙',
    destination: '日式舒芙蕾松饼与低卡果酱',
    topic: '云朵般云软口感，低糖低脂无负担。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：一口沦陷！云朵般绵软的舒芙蕾松饼，周末治愈系精致早餐！'
  },
  {
    id: 18,
    category: '旅游避坑',
    destination: '重庆洪崖洞与魔幻轻轨夜景拍摄定位',
    topic: '精准推荐最佳摄影打卡机位，避开收费假摄影师。',
    style: '干货硬核整理',
    core: '爆款标题：重庆洪崖洞夜景怎么拍出大片？本地摄影师私藏的5个免费机位！'
  },
  {
    id: 19,
    category: '数码科技',
    destination: '智能手环健康监测与打工人睡眠改善',
    topic: '分析深睡时间、心率预警与坐姿提醒功能。',
    style: '爆款种草测评',
    core: '爆款标题：熬夜打工人必备！智能手环深度体验，终于看清我的睡眠真面目！'
  },
  {
    id: 20,
    category: '穿搭生活',
    destination: '美式复古工装裤与高街街头穿搭',
    topic: '拉长腿部线条，匹配马丁靴与短款卫衣。',
    style: '爆款种草测评',
    core: '爆款标题：遮肉又显辣！美式复古工装裤搭配公式，酷女孩闭眼冲！'
  },
  {
    id: 21,
    category: '美妆护肤',
    destination: '眼周抗皱紧致眼霜与手法按摩',
    topic: '解决细网纹、黑眼圈与眼肿，配合淋巴按摩。',
    style: '干货硬核整理',
    core: '爆款标题：熬夜党告别黑眼圈！眼霜配合3步按摩手法，淡化细纹神清气爽！'
  },
  {
    id: 22,
    category: '美食烘焙',
    destination: '韩式芝士年糕辣火锅',
    topic: '家常调料比例，复刻韩剧同款沸腾美味。',
    style: '爆款种草测评',
    core: '爆款标题：追剧必备！自制韩式芝士年糕火锅，酱汁浓郁好吃到连汤都不剩！'
  },
  {
    id: 23,
    category: '旅游避坑',
    destination: '西北大环线自备物品与防晒保湿清单',
    topic: '高海拔高干燥气候下的护肤与穿搭物资汇总。',
    style: '干货硬核整理',
    core: '爆款标题：去西北大环线带什么？老驴友总结的行李清单，少带一样都受罪！'
  },
  {
    id: 24,
    category: '数码科技',
    destination: '无线机械键盘客制化轴体打字体验',
    topic: '声音清脆如麻将音，打字舒适，提高生产力。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：沉浸式听音解压！极客私藏麻将音机械键盘，桌面美学颜值担当！'
  },
  {
    id: 25,
    category: '穿搭生活',
    destination: '职场通勤托特包与容量测评',
    topic: '装得下14寸电脑、水杯与化妆包，牛皮耐磨。',
    style: '爆款种草测评',
    core: '爆款标题：打工人最爱的大容量托特包！能装又百搭，通勤背它绝不出错！'
  },
  {
    id: 26,
    category: '美妆护肤',
    destination: '定妆喷雾三步成膜法与夏季防汗',
    topic: '三合一定妆法，戴口罩不沾面，超长续航。',
    style: '干货硬核整理',
    core: '爆款标题：底妆待机一整天！三层定妆喷雾法，高温暴汗也不脱妆！'
  },
  {
    id: 27,
    category: '美食烘焙',
    destination: '低卡低脂燕麦无糖抹茶曲奇饼干',
    topic: '用燕麦粉替代面粉，口感酥脆香浓。',
    style: '极简图文金句',
    core: '爆款标题：减脂期零食自由！自制低卡燕麦抹茶曲奇，解馋无负担！'
  },
  {
    id: 28,
    category: '旅游避坑',
    destination: '杭州西湖隐秘游与避开人流的灵隐寺茶园',
    topic: '清晨看湖光山色，品龙井新茶，享受宁静。',
    style: '沉浸感日常 Vlog',
    core: '爆款标题：避开99%游客的杭州慢游路线！清晨灵隐茶园，呼吸自然的清香！'
  },
  {
    id: 29,
    category: '数码科技',
    destination: '桌面整理神器与隐蔽线材走线收纳',
    topic: '打造整洁无线桌面，理线盒与插排挂架测评。',
    style: '干货硬核整理',
    core: '爆款标题：告别蜘蛛网桌面！超强走线收纳干货，还你极简干净工作台！'
  },
  {
    id: 30,
    category: '穿搭生活',
    destination: '温柔系针织开衫与叠穿高领衬衫',
    topic: '柔软软糯质感，搭配香槟色百褶裙优雅有风情。',
    style: '极简图文金句',
    core: '爆款标题：穿上秒变温柔学姐！针织开衫叠穿公式，软糯糯的谁能不爱！'
  }
];

const samples = ref<XiaohongshuSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.destination.includes(searchQuery.value) ||
      s.style.includes(searchQuery.value) || 
      s.core.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
