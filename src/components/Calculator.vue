<template>
  <div class="calculator">
    <h1>dB计算器喵</h1>
    <el-form :model="form" label-width="180px">
      <el-form-item label="灵敏度 (dB/mW)">
        <el-input
          v-model.number="form.sensitivity"
          type="number"
          placeholder="请输入灵敏度"
        />
        <el-radio-group v-model="form.sensitivityUnit" class="unit-radio-group" @change="calculate">
          <el-radio label="dB/mW">dB/mW</el-radio>
          <el-radio label="dB/W">dB/W</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="阻抗 (Ω)">
        <el-input
          v-model.number="form.impedance"
          type="number"
          placeholder="请输入阻抗"
        />
        <el-radio-group v-model="form.impedanceUnit" class="unit-radio-group" @change="calculate">
          <el-radio label="Ω">Ω</el-radio>
          <el-radio label="kΩ">kΩ</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="响度 (dB)">
        <el-input
          v-model.number="form.loudness"
          type="number"
          placeholder="请输入响度"
        />
        <el-radio-group v-model="form.loudnessUnit" class="unit-radio-group" @change="calculate">
          <el-radio label="dB">dB</el-radio>
          <el-radio label="dB SPL">dB SPL</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="声压级 (Pa)">
        <el-input
          v-model.number="form.soundPressureLevel"
          type="number"
          placeholder="请输入声压级"
        />
        <el-radio-group v-model="form.soundPressureUnit" class="unit-radio-group" @change="calculate">
          <el-radio label="Pa">Pa</el-radio>
          <el-radio label="μPa">μPa</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item>
        <!-- 按钮禁用状态 -->
        <el-button 
          type="primary" 
          class="small-button" 
          @click="calculate" 
          :disabled="isCalculating"
        >
          计算
        </el-button>
      </el-form-item>
    </el-form>

    <div v-if="results" class="results">
      <p>所需功率: {{ results.power.toFixed(2) }} mW</p>
      <p>所需电压: {{ results.voltage.toFixed(2) }} V RMS</p>
      <p>所需电流: {{ results.current.toFixed(2) }} mA RMS</p>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from "vue";
import { ElForm, ElFormItem, ElInput, ElRadioGroup, ElRadio, ElButton } from "element-plus";

export default defineComponent({
  name: "Calculator",
  components: {
    ElForm,
    ElFormItem,
    ElInput,
    ElRadioGroup,
    ElRadio,
    ElButton
  },
  setup() {
    const form = ref({
      sensitivity: 98, // 默认灵敏度 dB/mW
      impedance: 32, // 默认阻抗 Ω
      loudness: 110, // 默认响度 dB
      soundPressureLevel: 0.00002, // 默认声压级 (Pa)
      sensitivityUnit: "dB/mW", // 默认灵敏度单位
      impedanceUnit: "Ω", // 默认阻抗单位
      loudnessUnit: "dB", // 默认响度单位
      soundPressureUnit: "Pa", // 默认声压级单位
    });

    const results = ref<{ power: number; voltage: number; current: number } | null>(null);
    const isCalculating = ref(false); // 防止重复计算

    const calculate = () => {
      if (isCalculating.value) return; // 防止重复计算

      isCalculating.value = true; // 计算开始时禁用按钮

      // 仅在单位转换时进行一次处理
      let impedance = form.value.impedance;
      if (form.value.impedanceUnit === "kΩ") {
        impedance *= 1000; // 将阻抗从kΩ转换为Ω
      }

      // 如果灵敏度单位为dB/W, 将其转换为dB/mW
      let sensitivity = form.value.sensitivity;
      if (form.value.sensitivityUnit === "dB/W") {
        sensitivity -= 30; // dB/mW 和 dB/W 差 30dB
      }

      // 如果响度单位为dB SPL, 将其转换为dB
      let loudness = form.value.loudness;
      if (form.value.loudnessUnit === "dB SPL") {
        loudness -= 94; // dB SPL 和 dB 的差值是 94dB
      }

      // 如果声压级单位为μPa，将其转换为Pa
      let soundPressureLevel = form.value.soundPressureLevel;
      if (form.value.soundPressureUnit === "μPa") {
        soundPressureLevel /= 1000000; // 将 μPa 转换为 Pa
      }

      let power = Math.pow(10, (loudness - sensitivity) / 10);
      let voltage = Math.sqrt(power * impedance);
      let current = voltage / impedance * 1000;

      results.value = { power, voltage, current };

      isCalculating.value = false; // 计算完成后重新启用按钮
    };

    // 监听敏感度、响度、阻抗和声压级的单位变化，自动触发计算
    watch(
      () => [
        form.value.sensitivityUnit,
        form.value.loudnessUnit,
        form.value.impedanceUnit,
        form.value.soundPressureUnit,
      ],
      () => {
        results.value = null; // 清除之前的结果
        calculate(); // 触发重新计算
      },
      { immediate: true } // 初始化时触发一次计算
    );

    return {
      form,
      results,
      calculate,
      isCalculating,
    };
  },
});
</script>

<style scoped>
.calculator {
  display: flex;
  justify-content: space-between; /* 使内容横向排列 */
  max-width: 1000px; /* 最大宽度，确保横向布局不会过于压缩 */
  margin: 0 auto;
  padding: 2rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  font-family: Arial, sans-serif;
  background-image: url('YOUR_IMAGE_URL_HERE'); /* 在这里插入背景图片URL */
  background-size: cover;
  background-position: center;
  gap: 20px; /* 添加间隔，避免元素紧凑 */
  flex-wrap: wrap; /* 允许元素在小屏幕上换行 */
}

.el-form {
  flex: 1; /* 让表单项占满可用空间 */
}

.unit-radio-group {
  display: flex;
  gap: 10px;
}

.results {
  margin-top: 1rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: rgba(249, 249, 249, 0.8); /* 添加背景色以增强文本可读性 */
  flex: 0.3; /* 固定结果区域宽度 */
}

.el-button {
  width: 100%;
}

/* 自定义按钮大小 */
.small-button {
  padding: 8px 16px; /* 调整按钮的内边距 */
  font-size: 14px; /* 调整字体大小 */
  height: 36px; /* 设置按钮高度 */
}

/* 手机端（小于768px） */
@media (max-width: 768px) {
  .calculator {
    flex-direction: column; /* 在手机上使用竖直排列 */
    align-items: center; /* 居中对齐 */
  }

  .results {
    flex: 0; /* 取消结果区宽度限制，适应竖排 */
    margin-top: 1rem;
    width: 100%; /* 使结果区域填充可用空间 */
  }

  .el-form {
    width: 100%; /* 确保表单区域宽度占满 */
  }
}
</style>
