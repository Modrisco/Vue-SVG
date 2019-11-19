<template>
  <div class="hello">
    <h2>Input field</h2>
    <label>
      <textarea v-model="commands" :placeholder="[[ hint ]]"></textarea>
    </label>
    <h3>Message: {{ errorMsg }}</h3>
<!--    <p>Number of lines: {{ linecount }}</p>-->
    <svg>
      <rect :x="[[ rectX ]]" :y="[[ rectY ]]" :width="[[ rectWidth ]]" :height="[[ rectHeight ]]"
            :style="{'fill': rectColor}"/>
      <circle :cx="[[ circleCx ]]" :cy="[[ circleCy ]]" :r="[[ circleR ]]" :style="{'fill': circleColor}"/>
      <polygon :points="[[ polyPoints ]]"
               :style="{'fill': polyColor}"/>
      <ellipse :cx="[[ ellipseCx ]]" :cy="[[ ellipseCy ]]" :rx="[[ ellipseRx ]]" :ry="[[ ellipseRy ]]" :style="{'fill': ellipseColor}"/>
    </svg>
  </div>
</template>

<script>
    export default {
        name: 'HelloWorld',
        data() {
            return {
                hint: 'R <X Coordinate> <Y Coordinate> <Width> <Height> - Should Draw ' +
                    'a rectangle with the parameters marked onto the SVG\n' +
                    'C <CX Coordinate> <CY Coordinate> <Radius> - Should Draw a ' +
                    'circle with the parameters marked onto the SVG\n' +
                    'P <X1,Y1> <X2,Y2> <X3,Y3> ..... <Xn,Yn> - Should draw a polygon ' +
                    'onto the SVG with the points specified\n' +
                    'E <CX Coordinate> <CY Coordinate> <RX Radius> <RY Radius> - Should draw a ellipse ' +
                    'with the parameters marked onto the SVG',

                commands: '',
                errorMsg: '',
                // TODO: only exist one SVG element for each shape
                hasCircle: false,
                hasRect: false,
                hasPoly: false,
                // default SVG element parameters
                // rect
                rectX: 0,
                rectY: 0,
                rectWidth: 0,
                rectHeight: 0,
                rectColor: '',
                circleCx: 0,
                circleCy: 0,
                circleR: 0,
                circleColor: '',
                polyPoints: '',
                polyColor: '',
                ellipseCx: 0,
                ellipseCy: 0,
                ellipseRx: 0,
                ellipseRy: 0,
                ellipseColor: '',
            }
        },
        methods: {
            // return this.commands.length
            lineCount: function (text) {
                return text.length ? text.split(/\r\n|\r|\n/).length : 0;
            },
            // generate random color for SVG elements
            RandomColor: function (shape) {
                let r, g, b
                r = Math.floor(Math.random() * 256)
                g = Math.floor(Math.random() * 256)
                b = Math.floor(Math.random() * 256)
                if (shape === 'circle') {
                    this.circleColor = `rgb(${r},${g},${b})`
                } else if (shape === 'rect') {
                    this.rectColor = `rgb(${r},${g},${b})`
                } else if (shape === 'poly') {
                    this.polyColor = `rgb(${r},${g},${b})`
                } else if (shape === 'ellipse') {
                    this.ellipseColor = `rgb(${r},${g},${b})`
                }
            },
            // compiler for each line of commands
            compiler: function (i, line) {
                const shapes = ['r', 'R', 'c', 'C', 'p', 'P', 'e', 'E']
                const command = line.split(' ')
                if (shapes.includes(command[0])|| command[0] === '') {
                    const shape = command[0]
                    if (shape === 'c' || shape === 'C') {
                        this.circleBuilder(i, command)
                    } else if (shape === 'r' || shape === 'R') {
                        this.rectBuilder(i, command)
                    } else if (shape === 'p' || shape === 'P') {
                        this.polyBuilder(i, command)
                    } else if (shape === 'e' || shape === 'E') {
                        this.ellipseBuilder(i, command)
                    }
                } else {
                    this.errorMsg = "Line " + i+1  + " Shape doesn't exist"
                }
            },
            // function creating polygon
            polyBuilder: function (i, command) {
                // if (this.hasPoly === true) {
                //     this.errorMsg = "Line " + i+1  + " Already has a rectangle"
                // }
                if (this.polyColor === '') {
                    this.RandomColor("poly")
                }
                let points = []
                if (command.length <= 3) {
                    this.errorMsg = "Line " + i+1  + " At least three polygon points"
                }
                for (let j = 1; j < command.length; j++) {
                    const point = command[j].split(',')
                    if (point.length !== 2 || isNaN(point[0] || isNaN(point[1]))) {
                        this.errorMsg = "Line " + i+1  + " Polygon grammar error"
                        return
                    } else {
                        points.push(`${point[0]}, ${point[1]}`)
                    }
                }
                this.polyPoints = points.join(' ')
            },
            // function creating rectangle
            rectBuilder: function (i, command) {
                // if (this.hasRect === true) {
                //     this.errorMsg = "Line " + i+1  + " Already has a rectangle"
                // }
                if (this.rectColor === '') {
                    this.RandomColor("rect")
                }
                if (command.length !== 5) {
                    this.errorMsg = "Line " + i+1  + " Parameter number error"
                }
                for (let j = 1; j <= 4; j++) {
                    if (isNaN(command[j])) {
                        this.errorMsg = "Line " + i+1  + " Rectangle parameters should be number"
                        return
                    }
                }
                this.rectX = command[1]
                this.rectY = command[2]
                this.rectWidth = command[3]
                this.rectHeight = command[4]
            },
            // function creating circle
            circleBuilder: function (i, command) {
                // if (this.hasCircle === true) {
                //     this.errorMsg = "Line " + i+1  + " Already has a circle"
                // }
                if (this.circleColor === '') {
                    this.RandomColor("circle")
                }
                if (command.length !== 4) {
                    this.errorMsg = "Line " + i+1  + " Parameter number error"
                }
                for (let j = 1; j <= 3; j++) {
                    if (isNaN(command[j])) {
                        this.errorMsg = "Line " + i+1  + " Circle parameters should be number"
                        return
                    }
                }
                this.circleCx = command[1]
                this.circleCy = command[2]
                this.circleR = command[3]
                // this.hasCircle = true
            },
            // Bonus: function creating ellipse
            ellipseBuilder: function (i, command) {
                if (this.ellipseColor === '') {
                    this.RandomColor("ellipse")
                }
                if (command.length !== 5) {
                    this.errorMsg = "Line " + i+1  + " Parameter number error"
                }
                for (let j = 1; j <= 4; j++) {
                    if (isNaN(command[j])) {
                        this.errorMsg = "Line " + i+1  + " Ellipse parameters should be number"
                        return
                    }
                }
                this.ellipseCx = command[1]
                this.ellipseCy = command[2]
                this.ellipseRx = command[3]
                this.ellipseRy = command[4]
            },
        },
        watch: {
            commands: function (val) {
                let lines = this.lineCount(val)
                if (lines > 4) {
                    this.errorMsg = "Too many command lines"
                } else {
                    const lineList = val.split(/\r\n|\r|\n/)
                    this.errorMsg = ""
                    for (let i = 0; i < lines; i++) {
                        this.compiler(i, lineList[i])
                    }
                }
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  textarea {
    height: 100px;
    width: 600px;
  }

  svg {
    height: 250px;
    width: 250px;
  }
</style>
