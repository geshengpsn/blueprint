<script setup lang="ts">
import { WebViewer } from "@rerun-io/web-viewer";
import { nanoid } from "nanoid";
import { ref, onMounted } from "vue";
const rerun = ref<HTMLElement | null>(null);
const ws = ref<WebSocket | null>(null);
onMounted(() => {
  if (rerun.value) {
    const rrdUrl = "ws://localhost:4321";
    const viewer = new WebViewer();
    viewer.start(rrdUrl, rerun.value, { width: "80vw", height: "100vh" });

    ws.value = new WebSocket("ws://localhost:4322");
    ws.value.onopen = () => {
      console.log("connected");
    };
    ws.value.onmessage = (event) => {
      console.log(event.data);
    };
    ws.value.onclose = () => {
      console.log("disconnected");
    };
  }
});

const predefine_nodes = [
  {
    id: 'find-board',
    label: 'Find Board & Press Button',
  },
  {
    id: 'move-slider',
    label: 'Move Slider to Setpoints on Screen',
  },
  {
    id: 'plug-probe',
    label: 'Plug in Probe into Test Port',
  },
  {
    id: 'open-door',
    label: 'Open Door, Probe Circuit',
  },
  {
    id: 'wrap-cable',
    label: 'Wrap Cable, Replace Probe',
  },
  {
    id: 'press-stop',
    label: 'Press Stop Trial Button',
  },
]
const nodes = ref<{
  node: number;
  id: string;
}[]>([]);
const handle_remove = (index: number) => {
  nodes.value.splice(index, 1);
};
const handle_insert = (index: number, node: number) => {
  nodes.value.splice(index, 0, { node: node, id: nanoid() });
};
const handle_start = () => {
  console.log(nodes.value);
  if (ws.value) {
    ws.value.send(JSON.stringify(nodes.value));
  }
};
</script>

<template>
  <div class="flex" data-theme="dark" >
    <div ref="rerun"></div>
    <div class=" overflow-auto flex-1" style="height: 100vh;">
      
      <div class=" flex flex-col justify-center items-center " style="">
        <button class="btn btn-primary btn-lg m-4" @click="handle_start()">Start</button>

        <TransitionGroup name="list">
          <div v-for="(node, i) in nodes" :key="node.id" class="flex flex-col items-center">
            <div className="dropdown dropdown-right">
              <!-- <div tabIndex={0} role="button" className="btn m-1">Click</div> -->
              <div tabIndex={node} role="button" class="btn btn-circle btn-outline m-4" @click="">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                  stroke="currentColor" class="size-6">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
                </svg>
              </div>
              <ul tabIndex={node} className="dropdown-content menu bg-base-100 rounded-box z-[1] w-52 p-2 shadow">
                <li @click="handle_insert(i, 0)"><a>Find Board & Press Button</a></li>
                <li @click="handle_insert(i, 1)"><a>Move Slider to Setpoints on Screen</a></li>
                <li @click="handle_insert(i, 2)"><a>Plug in Probe into Test Port</a></li>
                <li @click="handle_insert(i, 3)"><a>Open Door, Probe Circuit</a></li>
                <li @click="handle_insert(i, 4)"><a>Wrap Cable, Replace Probe</a></li>
                <li @click="handle_insert(i, 5)"><a>Press Stop Trial Button</a></li>
              </ul>
            </div>
            <div class="bg-neutral rounded-2xl shadow-xl p-4 m-4 size-40 flex flex-col">
              <div class="">
                <span class="">{{ predefine_nodes[node.node].label }}</span>
              </div>

              <button class="btn btn-sm mt-auto bottom-0" @click="handle_remove(i)">remove</button>
            </div>
          </div>
        </TransitionGroup>

        <div className="dropdown dropdown-right">
          <!-- <div tabIndex={0} role="button" className="btn m-1">Click</div> -->
          <div tabIndex={node} role="button" class="btn btn-circle btn-outline m-4">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
              stroke="currentColor" class="size-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
            </svg>
          </div>
          <ul tabIndex={node} className="dropdown-content menu bg-base-100 rounded-box z-[1] w-52 p-2 shadow">
            <li @click="nodes.push({node: 0, id: nanoid()})"><a>Find Board & Press Button</a></li>
            <li @click="nodes.push({node: 1, id: nanoid()})"><a>Move Slider to Setpoints on Screen</a></li>
            <li @click="nodes.push({node: 2, id: nanoid()})"><a>Plug in Probe into Test Port</a></li>
            <li @click="nodes.push({node: 3, id: nanoid()})"><a>Open Door, Probe Circuit</a></li>
            <li @click="nodes.push({node: 4, id: nanoid()})"><a>Wrap Cable, Replace Probe</a></li>
            <li @click="nodes.push({node: 5, id: nanoid()})"><a>Press Stop Trial Button</a></li>
          </ul>
        </div>
        <div class="bg-neutral rounded-2xl shadow-xl p-4 m-4">
          <div>
            <span class=" text-xl font-bold">end</span>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.list-move, /* 对移动中的元素应用的过渡 */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* 确保将离开的元素从布局流中删除
  以便能够正确地计算移动的动画。 */
.list-leave-active {
  position: absolute;
}
</style>