<template>
  <div
    :id="id"
    :key="key"
    :class="[
      'cell',
      {
        'vertical-line': addLine(Number(row)),
        'horizontal-line': addLine(Number(key)),
        'cell-start': value !== '-',
      },
    ]"
  >
    {{ value === "-" ? "" : value }}
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import { Prop } from "vue-property-decorator";

@Options({})
export default class Cell extends Vue {
  @Prop({ type: String, required: true }) readonly id!: string;
  @Prop({ type: String, required: true }) readonly key!: string;
  @Prop({ type: String, required: true }) readonly value!: string;
  @Prop({ type: String, required: true }) readonly row!: string;

  addLine(ent: number): boolean {
    return ent === 2 || ent === 5;
  }
}
</script>

<style scoped>
.cell {
  width: 48px;
  height: 48px;
  border: 1px solid lightgray;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
}
.cell-start {
  background-color: whitesmoke;
}

.horizontal-line {
  border-bottom: 1px solid black;
}

.vertical-line {
  border-right: 1px solid black;
}
</style>
