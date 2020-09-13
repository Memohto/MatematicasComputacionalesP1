<template>
  <b-container fluid>
    <div id="form">
      <b-form @submit="onSubmit" @reset="onReset">
        <b-row>
          <b-col>
            <label for="string-input">Cadena de entrada:</label>
            <b-form-textarea
              id="string-input"
              v-model="inputString"
              placeholder="Escriba la cadena a reemplazar"
              rows="6"
              max-rows="6"
              required
            ></b-form-textarea>
          </b-col>
        </b-row>
        <br>
        <b-row>
          <b-col>
            <label for="regex-input">Expresión regular:</label>
            <b-input type="text" id="regex-input" v-model="inputRegex" required></b-input>
          </b-col>
          <b-col>
            <label for="replace-string-input">Cadena reemplazadora:</label>
            <b-input type="text" id="replace-string-input" v-model="replaceString"></b-input>
          </b-col>
        </b-row>
        <br>
        <b-button type="submit" v-b-modal.output-modal variant="primary">Calcular</b-button>
        <b-button type="reset" variant="danger" style="margin-left:10px;">Reset</b-button>
        <b-modal id="output-modal" size="lg" title="Respuesta:">
          <b-form-textarea id="answer" plaintext rows="6" :value="outputString">
          </b-form-textarea>
        </b-modal>
      </b-form>
    </div>
    
    <div id="footer" class="container-fluid">
      <span>Guillermo Tanamachi - A01631327</span>
    </div>
  </b-container>
  
</template>

<script>
export default {
  name: 'Form',
  data: () => {
    return {
      inputString: "",
      inputRegex: "",
      replaceString: "",
      outputString: ""
    }
  },
  methods: {
    onReset(evt) {
      evt.preventDefault();
      this.inputString = "";
      this.inputRegex = "";
      this.replaceString = "";
      this.outputString = "";
    },
    onSubmit(evt) {
      evt.preventDefault();
      let AFN = this.createAFN();
      // console.log(this.validateAFN(AFN, this.inputString));
      // Doble aputadores
      let stringsToReplace = []
      for(let i = 0; i < this.inputString.length; i++) {
        for(let j = i+1; j <= this.inputString.length; j++) {
          if(this.validateAFN(AFN, this.inputString.substring(i, j))) {
            stringsToReplace.push(this.inputString.substring(i, j));
            i = j-1;
            break;
          }
        }
      }
      this.outputString = this.inputString;
      stringsToReplace.forEach(r => {
        this.outputString = this.outputString.replace(r, this.replaceString);
      })
    },
    createAFN() {
      let splitedRegex = this.inputRegex.split("+"); 
      let states = [];
      let isClosure = false;

      for(let i = 0; i < splitedRegex.length; i++) {
        states.push([this.createState("final", {})]);
        for(let j = splitedRegex[i].length-1; j >= 0; j--) {
          if(isClosure) {
            states[i][states[i].length-1].links = {
              ...states[i][states[i].length-1].links,
              [splitedRegex[i][j]]: states[i][states[i].length-1].id
            }
            isClosure = false;
          } else {
            if(splitedRegex[i][j] !== '*') {
              let links = {[splitedRegex[i][j]]: states[i][states[i].length-1].id}
              states[i].push(this.createState(""+j, links))
            } else {
              isClosure = true;
            }
          }
        }
      }

      let AFN = [];
      states.forEach(route => {
        let mappedRoute = {};
        route.map(state => {
          mappedRoute[state.id] = state.links;
        })
        AFN.push(mappedRoute);
      });

      return AFN;
    },
    createState(_id,_links) {
      let state = {
        id: _id,
        links: _links
      }
      return state
    },
    validateAFN(AFN, string) {
      let currentState = null;
      let done, valid = false
      for(let i = 0; i < AFN.length; i++) {
        let route = AFN[i];
        let j = 0;
        currentState = Object.keys(route)[0];
        done = false;
        while(j < string.length && !done) {
          if(route[currentState][string[j]]) {
            currentState = route[currentState][string[j]];
            valid = currentState == 'final';
          } else {
            done = true;
            valid = false;
          }
          j++
        }
        if(valid) break;
      }
      return valid;
    }
  }
}
</script>

<style scoped>
#footer {
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 5em;
  background-color: rgba(0, 0, 0, 0.1);
  border-top: 1px dotted black;
  color: black;
}
#form {
  margin: 0 auto;
  margin-top: 30px;
  padding: 25px;
  width: 50%;
  height: 26rem;
  border-radius: 25px;
  border: 2px solid black;
  background-color:rgba(65, 255, 75, 0.35);
  box-shadow: 5px 10px 18px #888888;
}
</style>