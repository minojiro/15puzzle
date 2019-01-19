<template>
  <div class="container">
    <div class="block-list">
      <template v-for="row in blocks">
        <div class="block"
          @click="blockClickHandler(block)"
          v-for="block in row"
          :key="block">
          <div class="block-inner" :class="{'is-empty': !block}">
            <div class="block-text">{{block}}</div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game',
  data() {
    return {
      blocks: [
        [1, 2, 3, 4],
        [5, 6, 7, 8],
        [9, 10, 11, 12],
        [13, 14, 15, 0],
      ],
    };
  },
  computed: {
    isCompleted() {
      const arr = this.blocks.reduce((acc, cur) => acc.concat(cur), []);
      return !arr.filter((n, i) => n !== (i + 1) % 16).length;
    },
  },
  mounted() {
    this.shuffle();
  },
  methods: {
    blockClickHandler(block) {
      this.moveBlock(block);
      this.checkComplete();
    },
    arrayDeepCopy(a) {
      return Array.isArray(a) ? a.map(this.arrayDeepCopy) : a;
    },
    shuffle() {
      for (let i = 0; i < 100; i ++) {
        const targetBlock = ~~(Math.random() * 15) + 1;
        this.moveBlock(targetBlock);
      };
    },
    checkComplete() {
      if (this.isCompleted) alert('completed!');
    },
    moveBlock(block) {
      const emptyPos = this.getBlockPos(0);
      const blockPos = this.getBlockPos(block);
      const diffX = emptyPos.x - blockPos.x;
      const diffY = emptyPos.y - blockPos.y;
      const isMovable = (!diffX || !diffY) && !(!diffX && !diffY);

      if (!isMovable) return;

      const incrementX = diffX > 0 ? 1 : (diffX < 0 ? -1 : 0);
      const incrementY = diffY > 0 ? 1 : (diffY < 0 ? -1 : 0);
      const loopCount = Math.abs(diffX + diffY);

      let newBlocks = this.arrayDeepCopy(this.blocks);
      let prevBlock = block;
      for (let i = 1; i <= loopCount; i++) {
        const curX = incrementX * i + blockPos.x;
        const curY = incrementY * i + blockPos.y;
        newBlocks[curY][curX] = prevBlock;
        prevBlock = this.blocks[curY][curX];
      }
      newBlocks[blockPos.y][blockPos.x] = 0;
      this.blocks = newBlocks;
    },
    getBlockPos(searchBlock) {
      let r = null;
      this.blocks.forEach((row, y) => row.forEach((block, x) => {
        if (block === searchBlock) r = {y, x};
      }));
      return r;
    },
  },
}
</script>

<style scoped>
.block-list {
  padding: 2px;
  box-sizing: border-box;
  background: #efefef;
  display: flex;
  flex-wrap: wrap;
}

.block {
  width: 25%;
}

.block-inner {
  background: #fff;
  box-sizing: border-box;
  border: 2px solid #efefef;
  position: relative;
  line-height: 1;
}

.block-inner:before {
  content: '';
  display: block;
  padding-top: 100%;
}

.block-text {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  margin-top: -0.5em;
  text-align: center;
  font-size: 28px;
  font-weight: bold;
}

.block-inner.is-empty {
  background: #efefef;
  color: transparent;
}

.container {
  margin: 0 auto;
  max-width: 680px;
}
</style>