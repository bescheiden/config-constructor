<template>
  <div class="list-container">
    <div class="list1">
          <h6>#list1name</h6>
          <div class="text-area">
            <el-input type="textarea" :rows="4" placeholder="填写商品id,用逗号“,”隔开">
            </el-input>
          </div>
        </div>
  </div>
</template>
<script>
export default {
  name: "goodsList",
  props: ["listName"],
  data() {
    return {
    };
  },
  created() {
  },
  methods: {
  }
};
</script>

<style >
h6 {
  margin: 15px 0;
}
</style>
