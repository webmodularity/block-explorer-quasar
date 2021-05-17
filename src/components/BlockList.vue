<template>
  <div class="q-pa-sm">
    <q-ajax-bar
      ref="bar"
      position="top"
      color="accent"
      size="10px"
      skip-hijack
    />
    <div class="row">
      <div class="col-12">
        <q-markup-table>
          <thead>
          <tr>
            <th class="text-left">Block Number</th>
            <th class="text-right">Hash</th>
            <th class="text-right">Gas Used</th>
            <th class="text-right">Transactions</th>
            <th class="text-right">Difficulty</th>
            <th class="text-right">Miner</th>
            <th class="text-right">Mined</th>
          </tr>
          </thead>
          <tbody>

          <tr v-for="block in data">
            <td class="text-left">{{ block.number }}</td>
            <td class="text-right">{{ block.hash.slice(0,8) }}...{{ block.hash.slice(block.hash.length - 8) }}</td>
            <td class="text-right">{{ block.gasUsed.toNumber().toLocaleString('en-us') }}</td>
            <td class="text-right">{{ block.transactions.length }}</td>
            <td class="text-right">{{ block.difficulty.toLocaleString('en-us') }}</td>
            <td class="text-right">{{ block.miner.slice(0,8) }}...{{ block.miner.slice(block.miner.length - 8) }}</td>
            <td class="text-right">{{ humanizeDuration((Date.now() - (block.timestamp * 1000)), {round: true}) }} ago</td>
          </tr>
          </tbody>
        </q-markup-table>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <q-btn :loading="loading" class="full-width" color="primary" glossy v-on:click="loadData()">
        Refresh Block Data
        <template v-slot:loading>
          <q-spinner-hourglass class="on-left" />
          Loading Latest Blocks from Infura Node...
        </template>
        </q-btn>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { ethers } from "ethers";
const humanizeDuration = require("humanize-duration");

export default defineComponent({
  name: 'BlockList',
  setup () {
    const data = ref(null);
    const loading = ref(false);
    const blockCache = [];
    const providerUrl = 'https://mainnet.infura.io/v3/8c6ed301e95847798eeaf1a2dbf3d5ec';
    const provider = new ethers.providers.JsonRpcProvider(providerUrl);

    async function loadData () {
      console.time("loadBlocks");
      loading.value = true;

      const latestBlock = await provider.getBlock();
      const lastBlocks = [latestBlock];

      for (let i = 1;i < 10;i++) {
        const targetBlockNumber = latestBlock.number - i;
        if (blockCache[targetBlockNumber]) {
          lastBlocks.push(blockCache[targetBlockNumber]);
        } else {
          const fetchedBlock = await provider.getBlock(targetBlockNumber);
          lastBlocks.push(fetchedBlock);
          // Add to cache
          blockCache[targetBlockNumber] = fetchedBlock;
        }
      }
      data.value = lastBlocks;
      loading.value = false;
      console.timeEnd("loadBlocks");
    }
    return { data, loadData, humanizeDuration, loading }
  },
  mounted() {
    this.loadData();
  }
})
</script>

<style lang="sass" scoped>
.row > div
  padding: 10px 15px
  background: $grey-1
  border: 1px solid rgba(86,61,124,.2)
.row + .row
  margin-top: 1rem
</style>
