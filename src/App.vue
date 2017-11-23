<template>
  <div>
    <header>
      <nav class="navbar navbar-fixed-bottom">
        <div class="container">
          <div class="navbar-header">
            <a class="navbar-brand" href="">Memory game</a>
          </div>
          <form class="navbar-form navbar-left" role="search">
            <div class="form-group">
              <input type="text" placeholder="Name..." class="form-control" name="name" v-on:input="name = $event.target.value" :disabled="disabled" />
            </div>
            <button type="button" @click="startMethod()" :disabled="disabled || name == ''" class="btn btn-primary">Start!</button>
          </form>
          <div class="navbar-right">
            <p class="navbar-text">Time: <b>{{ this.time }}</b></p>
            <p class="navbar-text">Rounds: <b>{{ this.rounds }}</b></p>
            <p class="navbar-text"><button type="button" @click="restartMethod()" :disabled="!disabled" class="btn btn-sm btn-danger">Restart!</button></p>
          </div>
        </div>
      </nav>
    </header>
    <section class="container-fluid desk">
      <div class="letters container" v-if="!stopGame">
        <div class="letter" v-for="letter in letters" :class="{ turned: letter.turned, certain: letter.certain}" @click="turnLetter(letter)">
          <div class="back"></div>
          <div class="front" v-bind:style="{ backgroundImage: 'url(' + letter.img + ')' }"></div>
        </div>
      </div>
      <div class="info container" v-if="showResult">
        <div class="jumbotron" v-if="showFinalResult">
          <div class="well">
            <div class="page-header">
              <h2>Well done!!</h2>
            </div>
            <div class="alert alert-success" role="alert">Points: {{ rounds }}</div>
            <div class="form-group">
              <button @click="restartMethod()" class="btn btn-primary btn-block btn-lg">Play again!</button>
            </div>
          </div>
          <div class="resultados">
            <table class="table table-striped table-bordered">
              <thead>
                <tr>
                  <th>Player</th>
                  <th>Time</th>
                  <th>Rounds</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="player in orderResults" class="success">
                  <td>{{ player.name }}</td>
                  <td>{{ player.time }}</td>
                  <td>{{ player.rounds }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
<script>
const assetsPath = './src/assets';
let lettersNumber = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25];
lettersNumber = _.shuffle(lettersNumber);
let letters = [
  {img: assetsPath + '/letters/'+lettersNumber[0]+'.jpg', title: lettersNumber[0]},
  {img: assetsPath + '/letters/'+lettersNumber[1]+'.jpg', title: lettersNumber[1]},
  {img: assetsPath + '/letters/'+lettersNumber[2]+'.jpg', title: lettersNumber[2]},
  {img: assetsPath + '/letters/'+lettersNumber[3]+'.jpg', title: lettersNumber[3]},
  {img: assetsPath + '/letters/'+lettersNumber[4]+'.jpg', title: lettersNumber[4]},
  {img: assetsPath + '/letters/'+lettersNumber[5]+'.jpg', title: lettersNumber[5]},
  {img: assetsPath + '/letters/'+lettersNumber[6]+'.jpg', title: lettersNumber[6]},
  {img: assetsPath + '/letters/'+lettersNumber[7]+'.jpg', title: lettersNumber[7]},
  {img: assetsPath + '/letters/'+lettersNumber[8]+'.jpg', title: lettersNumber[8]},
  {img: assetsPath + '/letters/'+lettersNumber[9]+'.jpg', title: lettersNumber[9]}
];
const customShuffle = () => {
  let pack = [].concat(
    _.cloneDeep(letters),
    _.cloneDeep(letters)
  );
  return _.shuffle(pack);
}
export default {
  data() {
    return {
      name: '', stopGame: true, disabled: false, timewatch: 0, time: 0, letters: [],
      rounds: 0, combinations: 0, timeTurneOff: 0, showResult: false, showFinalResult: false,
      players: []
    }
	},
	
	methods: {
		startMethod() {
      this.showResult = false;
      this.showFinalResult = false;
      this.stopGame = false;
      this.disabled = true;
      this.timewatch = setInterval(() => {
          this.time++;
      }, 1000);
		},
    restartMethod() {
      this.showResult = false;
      this.showFinalResult = false;
      this.stopGame = true;
      this.disabled = false;
      this.time = 0;
      this.rounds = 0;
      clearInterval(this.timewatch);
      this.timewatch = 0;
      let letters = customShuffle();   
      _.each(letters, (letter) => {
          letter.turned = false;
          letter.certain = false;
      });     
      this.letters = letters;	
		},
    endMethod() {
      this.stopGame = true;
      clearInterval(this.timewatch);
      this.showResult = true;
      this.showFinalResult = true;

      let player = {
        name: this.name,
        time: this.time,
        rounds: this.rounds
      };
      this.players.push(player);
    },
    lettersTurned() {
      return _.filter(this.letters, letter => letter.turned);
    },
    sameLetterTurned() {
      let lettersTurned = this.lettersTurned();
      if (lettersTurned.length == 2) {
        if (lettersTurned[0].title == lettersTurned[1].title) {
          return true;
        }
      }
    },
    letLetterTurned() {
      _.each(this.letters, (letter) => {
        if (letter.turned) {
          letter.certain = true;
        }
      });
    },
    verifyIfCorrect() {
      let lettersFound = _.filter(this.letters, letter => letter.certain);
      this.combinations = lettersFound.length;
      if (lettersFound.length == this.letters.length) {
        return true;
      }

    },
    turnLetter(letter) {
      if (letter.certain || letter.turned) return;
      let count = this.lettersTurned().length;
      if (count == 0) {
          letter.turned = !letter.turned;
      } else if (count == 1) {
        letter.turned = !letter.turned;
        this.rounds++;
        if (this.sameLetterTurned()) {
          this.flipBackTimer = setTimeout( ()=> {
            this.clearTime();
            this.letLetterTurned();
            this.turnOff();
            if (this.verifyIfCorrect()) {
              this.endMethod();
            }   
          }, 100);
        } else {
          this.timeTurneOff = setTimeout( ()=> {
              this.turnOff();
              this.clearTime();
          }, 500);
        }
      }
    },
    clearTime() {
      clearTimeout(this.timeTurneOff);
      this.timeTurneOff = null;
    },
    turnOff() {
      _.map(this.letters, letter => letter.turned = false);
    },
	},
  computed: {
    orderResults: function() {
      function compare(a, b) {
        if (a.rounds < b.rounds) {
          return -1;
        }
        if (a.rounds > b.rounds) {
          return 1;
        }
        return 0;
      }
      return this.players.sort(compare);
    }
  },
	created() {
		this.restartMethod();
	}
}
</script>
<style scoped>
  * { box-sizing: border-box; }
  .desk {
    background-color: #2B3736;
    padding: 20px;
    min-height: 100%;
    display: block;
    position: absolute;
    width: 100%;
    height: 100%;
  }
  .letters .letter {
    display: inline-block;
    width: 10vw;
    height: 10vw;
    margin: 2px;
    transition: opacity 0.5s;
    position: relative;
  }
  .letters .letter .front, .letters .letter .back {
    border-radius: 10px;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    transform: translateZ(0);
    transition: transform 0.5s;
    transform-style: preserve-3d;
  }
  .letters .letter .back {
    background: #fefefe url('/src/assets/db1.png') no-repeat center;
    background-size: 50%;
  }
  .letters .letter .front {
    transform: rotateY(-180deg);
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
  }
  .letters .letter.turned .back, .letters .letter.certain .back {
    transform: rotateY(180deg);
  }
  .letters .letter.turned .front, .letters .letter.certain .front {
    transform: rotateY(0deg);
  }
  .letters .letter.certain {
    opacity: 0.1;
  }
  .navbar{
    background-color: #fefefe;
  }
  .navbar-text button{
    margin-top: -5px
  }
</style>
