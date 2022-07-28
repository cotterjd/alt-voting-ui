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

<script>
import Hello from '../components/HelloWorld.vue'
import series from 'promise.series'
import * as R from 'ramda' // TODO: import these individually

export default {
  components: {
    Hello,
  },
  data: () => ({
    winner: ``,
    votes: {},
    countObj: {},
    losers: [],
    VoteString: ``,
  }),
  mounted () {
    const initialVotes = {
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
    calcVotes (votesObj) {
      const topVotes = getTopVotes(votesObj)
      const losers = getLowest(topVotes)
      const newLosers = R.uniq([...this.losers, ...losers])
      this.losers = newLosers
      return Object.keys(votesObj).reduce((acc, voter) => ({ ...acc, [voter]: getUpdatedVoterPicks(newLosers, votesObj[voter])}), {})
    },
    startVote () {
      if (!this.winner) {
        const newVotes = this.calcVotes(this.votes)
        // TODO: clean this up
        const topVotes = getTopVotes(newVotes)
        const winner = getWinner(topVotes, topVotes.length)
        if (winner) this.winner = winner
        this.votes = newVotes
        const countObj = getCountObj(newVotes)
        this.countObj = countObj
      }
    },
    // {} -> `` 
  }
}
function getUpdatedVoterPicks (losers, voterPicks) {
  if (!losers.includes(voterPicks[0])) return voterPicks
  return getUpdatedVoterPicks(losers, voterPicks.slice(1))
}
// {} -> {} 
function getCountObj (votesObj) {
  // [``]
  const topVotes = getTopVotes(votesObj)
  // {}
  const countObj = topVotes.reduce((acc, str) => {
    if (acc[str]) return { ...acc, [str]: acc[str]+1 }
    return { ...acc, [str]: 1 }
  }, {})
  return countObj
}
// {} -> [``]
function getTopVotes (votesObj) {
  return Object.keys(votesObj).map(key => votesObj[key][0])
}
// [[``]] -> int -> ``
export function getWinner (votes, totalVotes) {
  const sortFunc = (a, b) => a === b ? -1 : 1 
  // [[]]
  const grouped = R.groupWith(R.equals, R.sort(sortFunc, votes))
  const highest = grouped.reduce((high, supporters) => supporters.length > high.length ? supporters : high, grouped[0]) 
  const lengthNeeded = Math.floor(totalVotes / 2 + 1)
  return highest?.length >= lengthNeeded ? highest?.[0] : ``
}
// [``] -> [] 
function getLowest (votes) {
  const sortFunc = (a, b) => a === b ? -1 : 1 
  // [[]]
  const grouped = R.groupWith(R.equals, R.sort(sortFunc, votes))
  // [``]
  const losers = R.flatten(grouped.reduce((acc, supporters) => {
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
