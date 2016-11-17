<template>
  <div id="app">
    <div class="row">
      <div class="col-xs-12">
        <h1>linear-congruences</h1>
        <h3>
          by <a href="https://github.com/malonehedges">@malonehedges</a>
        </h3>
      </div>
    </div>

    <div class="row">
      <div class="col-xs-12">
        <label>
          a <input type="number" v-model.number="a" />
        </label>
        <label>
          m <input type="number" v-model.number="m" />
        </label>
      </div>
    </div>

    <div v-if="gcd !== 1" class="row">
      <div class="col-xs-12">
        <h2>gcd({{ a }}, {{ m }}) = {{ gcd }} ≠ 1</h2>
      </div>
    </div>

    <div v-if="gcd === 1" class="row">
      <div class="col-xs-12 col-sm-6">
        <h3>1. Euclidian Algorithm</h3>
        <div v-for="step in euclidianSteps">
          <strong>{{ step[0] }}</strong> = {{ step[1] }}(<strong>{{ step[2] }}</strong>) + {{ step[3] }}
        </div>
      </div>
      <div class="col-xs-12 col-sm-6">
        <h3>2. Back Substitution</h3>
        <div v-for="step in remainderSteps">
          {{ step[0] }} = {{ step[1]}} + {{ step[2] }}({{ step[3] }})
        </div>
      </div>
      <div class="col-xs-12 col-sm-12">
        <h3>3. Solve</h3>
        <div v-for="step in solveSteps">
          1
          = {{ step[0][0] }}({{ step[0][1] }}) + {{ step[0][2] }}({{ step[0][3] }} + {{ step[0][4] }}({{ step[0][5] }}))
          = <strong>{{ step[1][0] }}</strong>({{ step[1][1] }}) + <strong>{{ step[1][2] }}</strong>({{ step[1][3] }})
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',

  data () {
    return {
      // Example input
      a: 19,
      m: 141,

      // Computed data
      gcd: 1,
      euclidianSteps: [],
      remainderSteps: [],
      solveSteps: []
    }
  },

  // Solve when ready
  created () {
    this.compute()
  },

  // Using watch instead of event handlers.
  // While event handlers in the template are better for most scenarios,
  // the number input type is strange with changed values.
  watch: {
    m: {
      handler () {
        this.compute()
      }
    },
    a: {
      handler () {
        this.compute()
      }
    }
  },

  methods: {
    reset () {
      this.euclidianSteps = []
      this.remainderSteps = []
      this.solveSteps = []
    },

    compute () {
      // a must be smaller than m
      if (this.a > this.m) {
        return this.reset()
      }

      this.euclidian()

      // This method doesn't work with non-1 gcd values
      if (this.gcd !== 1) {
        return this.reset()
      }

      this.remainders()
      this.solve()
    },

    euclidian () {
      let steps = []

      let currentA = this.a
      let currentM = this.m

      while (currentA > 1) {
        let mult = Math.floor(currentM / currentA)
        let extra = currentM - (mult * currentA)

        // m = (mult)*a + extra
        steps.push([currentM, mult, currentA, extra])

        // shift things over for the next iteration
        currentM = currentA
        currentA = extra
      }

      // if currentA is 0, use currentM
      this.gcd = currentA || currentM
      this.euclidianSteps = steps
    },

    remainders () {
      // Use to copy the array, as `Array.reverse` does it directly to the array.
      let steps = Array.prototype.slice.call(this.euclidianSteps)

      for (let i = 0; i < steps.length; i++) {
        let step = steps[i]

        // 1 = 22 - 7*3
        // Note, while we could multiply step[1] by `-1` to have the array be accurate,
        // it is left this way for easier formatting in the page

        // maybe not...
        steps[i] = [step[3], step[0], -1 * step[1], step[2]]
      }

      this.remainderSteps = steps
    },

    solve () {
      // Use to copy the array, as `Array.reverse` does it directly to the array.
      let steps = Array.prototype.slice.call(this.remainderSteps)

      // Work from bottom to top
      steps.reverse()

      // Finish early if there's nothing to be done
      if (steps.length === 1) {
        let step = steps[0]
        this.solveSteps = [
          [
            // i.e. 1 = 1(141) + -7(20 + 0(0))
            // TODO: Leave this part out entirely when there's just one equation
            // i.e. 1 = 1(141) + -7(20)
            [ 1, step[1], step[2], step[3], 0, 0 ],
            step
          ]
        ]
        return
      }

      // Each step has 2 parts:
      //   1. initial equation (6 elements)
      //   2. simplified equation (4 elements)
      // solveSteps = [ [ [|6|], [|4|] ], ... ]
      let solveSteps = []

      for (let i = 0; i < steps.length - 1; i++) {
        let step = steps[i]
        let nextStep = steps[i + 1]

        let intermediateStep

        // "Don't Repeat Yourself"
        if (i === 0) {
          intermediateStep = [
            1,
            step[1],
            step[2],
            nextStep[1],
            nextStep[2],
            nextStep[3]
          ]
        } else {
          let previousSolution = solveSteps[i - 1][1]
          intermediateStep = [
            previousSolution[0],
            previousSolution[1],
            previousSolution[2],
            nextStep[1],
            nextStep[2],
            nextStep[3]
          ]
        }

        let currentStep = [
          intermediateStep[2],
          intermediateStep[3],
          intermediateStep[0] + (intermediateStep[2] * intermediateStep[4]),
          intermediateStep[5]
        ]

        // TODO: Explain logic happening here, and why special case is needed

        solveSteps.push([
          intermediateStep,
          currentStep
        ])
      }

      this.solveSteps = solveSteps
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #253452;
  margin-top: 60px;
}
</style>
