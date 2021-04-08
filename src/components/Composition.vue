<template>
  <button>count is: {{ count }}</button>
  <p>{{doublerCount}}</p>
  <p>{{ msg }}</p>
</template>

<script>
import { reactive, computed, onMounted, onUnmounted, ref, toRefs } from "vue";
export default {
  name: "Composition",
  props: {
    msg: String
  },
  setup() {
    let { count, doublerCount } = userCount();
    const msg = ref([1]); //ref 可以定义一个可以响应的变量
    console.log();
    msg.value.push(2);
    return { count, doublerCount, msg };
  }
};
function userCount() {
  const data = reactive({
    count: 1,
    doublerCount: computed(() => data.count * 2)
  });

  let timer;
  onMounted(() => {
    timer = setInterval(() => {
      data.count++;
    }, 1000);
  });

  onUnmounted(() => {
    clearInterval(timer);
  });
  return toRefs(data); //当从合成函数返回响应式对象时，toRefs 非常有用，这样消费组件就可以在不丢失响应性的情况下对返回的对象进行分解/扩散：
}
</script>
