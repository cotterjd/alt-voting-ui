<template>
  <div>
    <button :v-show="!winner" @click="startVote()">Simulate Next Round</button>
    <div style="display: flex">
      <hello v-for="key in Object.keys(votes)" :key="key" :voter="key" :picks="votes[key]" />
      <div>
        <div v-for="key in Object.keys(countObj)" :key="key">{{key}}: {{countObj[key]}}</div>
        <div v-for="loser in losers" :key="loser"><s>{{loser}}</s></div>
      </div>
    </div>
    <h1>{{ winner ? `Winner: ${winner}` : ``}}</h1>
  </div>
</template>

<script lang="ts">
import Hello from '../components/HelloWorld.vue'
import * as R from 'ramda' // TODO: import these individually

interface Data {
  winner: string
  votes: Record<string, string[]>
  countObj: Record<string, number>
  losers: string[]
}
export default {
  components: {
    Hello,
  },
  data: (): Data => ({
    winner: ``,
    votes: {},
    countObj: {},
    losers: [],
  }),
  mounted () {
    const initialVotes: Record<string, string[]> = {
      joe: [`Republican`, `Democrat`, `Libertarian`, `Green Party`],
      paul: [`Green Party`, `Libertarian`, `Republican`, `Democrat`],
      dan: [`Republican`, `Libertarian`, `Democrat`, `Green Party`],
      steve: [`Libertarian`, `Republican`, `Green Party`, `Democrat`],
      pauline: [`Democrat`, `Green Party`, `Republican`, `Libertarian`],
      jessica: [`Green Party`, `Democrat`, `Libertarian`, `Republican`],
      bob: [`Green Party`, `Republican`, `Democrat`, `Libertarian`],
    }
    this.votes = initialVotes
    this.countObj = getCountObj(initialVotes) 
  },
  computed: {},
  methods: {
    // TODO: fix not stopping after majority
    calcVotes (votesObj: Record<string, string[]>) {
      const topVotes: string[] = getTopVotes(votesObj)
      const losers: string[] = getLowest(topVotes)
      const newLosers: string[] = R.uniq([...this.losers, ...losers])
      this.losers = newLosers
      return Object.keys(votesObj).reduce((acc, voter) => ({ ...acc, [voter]: getUpdatedVoterPicks(newLosers, votesObj[voter])}), {})
    },
    startVote () {
      if (!this.winner) {
        const newVotes: Record<string, string[]> = this.calcVotes(this.votes)
        // TODO: clean this up
        const topVotes: string[] = getTopVotes(newVotes)
        const winner: string = getWinner(topVotes, topVotes.length)
        if (winner) this.winner = winner
        this.votes = newVotes
        const countObj = getCountObj(newVotes)
        this.countObj = countObj
      } 
    },
  }
}
function getUpdatedVoterPicks (losers: string[], voterPicks: string[]): string[] {
  if (!losers.includes(voterPicks[0])) return voterPicks
  return getUpdatedVoterPicks(losers, voterPicks.slice(1))
}
function getCountObj (votesObj: Record<string, string[]>): Record<string, number> {
  const topVotes: string[] = getTopVotes(votesObj)
  const countObj: Record<string, number> = topVotes.reduce((acc, str) => {
    if (acc[str]) return { ...acc, [str]: acc[str]+1 }
    return { ...acc, [str]: 1 }
  }, {})
  return countObj
}
function getTopVotes (votesObj: Record<string, string[]>): string[] {
  return Object.keys(votesObj).map(key => votesObj[key][0])
}
function sortFunc (a: string, b: string): number {
  return a === b ? -1 : 1 
}
export function getWinner (votes: string[], totalVotes: number): string {
  const grouped: string[][] = R.groupWith(R.equals, R.sort(sortFunc, votes))
  const highest: string[] = grouped.reduce((high, supporters) => supporters.length > high.length ? supporters : high, grouped[0]) 
  const lengthNeeded: number = Math.floor(totalVotes / 2 + 1)
  return highest?.length >= lengthNeeded ? highest?.[0] : ``
}
function getLowest (votes: string[]): string[] {
  const grouped: string[][] = R.groupWith(R.equals, R.sort(sortFunc, votes))
  const losers: string[] = R.flatten(grouped.reduce((acc, supporters) => {
    if (supporters.length === acc[0].length) return R.uniq([...acc, supporters])
    if (supporters.length < acc[0].length) return [supporters]
    return acc
  }, [grouped[0]]))
  return losers
}
</script>

<style>
  body {
    font-family: sans-serif;
    padding: 10px;
  }
</style>
