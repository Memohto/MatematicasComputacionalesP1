<template>
  <b-container fluid>
    <div id="form">
      <b-form @submit="onSubmit" @reset="onReset">
        <b-row>
          <b-col>
            <b-button class="float-right" v-b-toggle.sidebar-1>
              <span>Info</span>
            </b-button>
          </b-col>
        </b-row>
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
        <b-button type="submit" variant="primary">Calcular</b-button>
        <b-button type="reset" variant="danger" style="margin-left:10px;">Reset</b-button>
        <b-modal id="output-modal" cancel-title='Reset' size="lg" title="Respuesta:" @cancel='resetModal'>
          <b-form-textarea id="answer" plaintext rows="6" :value="outputString">
          </b-form-textarea>
        </b-modal>
        <b-sidebar id="sidebar-1" title="Sidebar" shadow>
          <div class="px-3 py-2">
            <p>
              Cras mattis consectetur purus sit amet fermentum. Cras justo odio, dapibus ac facilisis
              in, egestas eget quam. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.
            </p>
            <b-img src="https://picsum.photos/500/500/?image=54" fluid thumbnail></b-img>
          </div>
        </b-sidebar>
      </b-form>
    </div>
    
    <div id="footer" class="container-fluid">
      <b-row>
        <b-col style="margin-top: 15px">
          <img alt="Tec logo" src="../assets/logo.png" style="width: 150px; height: 50px">
        </b-col>
        <b-col style="text-align: right;">
          <p>
            <span style="display: block;">Guillermo Hiroshi Tanamachi Ortega - A01631327</span>
            <span style="display: block;">Matemáticas Computacionales: Proyecto Parcial 1</span>
            <span style="display: block;">Profesor: Luis Ricardo Peña Llamas</span>
          </p>
        </b-col>
      </b-row>
      
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
    resetModal() {
      this.inputString = "";
      this.inputRegex = "";
      this.replaceString = "";
      this.outputString = "";
    },
    onSubmit(evt) {
      evt.preventDefault();
      let AFN = this.createAFN();

      let substrings = this.getAllSubstrings(this.inputString);
      let substringsToReplace = substrings
        .map(s => {
          if(this.validateAFN(AFN, s)) {
            return s;
          }        
        })
        .filter(Boolean)
        .sort((a, b) => {
          return b.length-a.length;
        });

      this.outputString = this.inputString;
      substringsToReplace.forEach(r => {
        this.outputString = this.outputString.replace(r, this.replaceString);
      });

      this.$bvModal.show('output-modal');
    },
    getAllSubstrings(string) {
      let substrings = [];
      for (let i = 0; i < string.length; i++) {
          for (let j = i + 1; j <= string.length; j++) {
              substrings.push(string.slice(i, j));
          }
      }
      return substrings;
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
  background-color: rgba(0, 0, 0, 0.2);
  border-top: 1px dotted black;
  color: black;
  z-index: -100;
}
#form {
  margin: 0 auto;
  margin-top: 20px;
  padding: 25px;
  width: 50%;
  height: 28rem;
  border-radius: 25px;
  border: 2px solid black;
  background-color:rgba(65, 255, 75, 0.35);
  box-shadow: 5px 10px 18px #888888;
}
</style>