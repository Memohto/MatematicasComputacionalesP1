<template>
  <b-container fluid>
    <div id="form">
      <b-form @submit="onSubmit">
        <b-row>
          <b-col>
            <label for="string-input">Cadena de entrada:</label>
            <b-form-textarea
              id="string-input"
              v-model="inputString"
              placeholder="Escriba su cadena"
              rows="6"
              max-rows="6"
            ></b-form-textarea>
          </b-col>
        </b-row>
        <br>
        <b-row>
          <b-col>
            <label for="regex-input">Expresión regular:</label>
            <b-input type="text" id="regex-input" v-model="inputRegex"></b-input>
          </b-col>
          <b-col>
            <label for="replace-string-input">Cadena reemplazadora:</label>
            <b-input type="text" id="replace-string-input" v-model="replaceString"></b-input>
          </b-col>
        </b-row>
        <br>
        <!-- <b-row>
          <b-col>
            <label for="string-input">Cadena de salida:</label>
            <b-form-textarea
              id="string-output"
              v-model="outputString"
              rows="2"
              max-rows="3"
            ></b-form-textarea>
          </b-col>
        </b-row>
        <br> -->
        <b-button type="submit" variant="primary">Calcular</b-button>
      </b-form>
    </div>
    
    <div id="footer" class="container-fluid">
      <span>Guillermo Tanamachi - A01631327</span>
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
    onSubmit(evt) {
      evt.preventDefault();
      let AFN = this.createAFN();
      this.validateAFN(AFN, this.inputString);
    },
    createAFN() {
      let initialState = this.createState("init", {});
      let finalState = this.createState("final", {});
      let states = [finalState];
      let splitedRegex = this.inputRegex.split("+"); 
      let stateNumber;
      let isClosure = false;
      a: for(let i = 0; i < splitedRegex.length; i++) {
        if(splitedRegex[i][0] == '*') {
          console.log("Error: No se puede iniciar la regex con '*'");
          break;
        }
        stateNumber = 0;
        for(let j = splitedRegex[i].length-1; j >= 0; j--) {
          console.log(splitedRegex[i][j]);
          if(isClosure) {
            if(splitedRegex[i][j] !== '*') {
              if(j == splitedRegex[i].length-2) {
                if(splitedRegex[i].length == 2) {
                  states[states.length-1].links = {
                    ...states[states.length-1].links,
                    [splitedRegex[i][j]]: 'final'
                  } 
                  initialState.links = {
                    ...initialState.links,
                    [splitedRegex[i][j]]: 'final'
                  } 
                } else {
                  states.forEach(s => {
                    if(s.id == 'final') {
                      s.links = {
                        ...s.links,
                        [splitedRegex[i][j]]: 'final'
                      } 
                    }
                  })
                }
              } else if(j == 0) {
                console.log("cerradura izquieda")
                Object.keys(states[states.length-1].links).forEach(l => {
                  console.log(states[states.length-1].links[l]);
                  console.log(states[states.length-1].id);
                  if(states[states.length-1].links[l] == states[states.length-1].id) {
                    states[states.length-1].links[l] = 'init';
                  }
                })
                states[states.length-1].id = 'init';
                states[states.length-1].links = {
                  ...states[states.length-1].links,
                  [splitedRegex[i][j]]: 'init'
                }
              } else {
                states[states.length-1].links = {
                  ...states[states.length-1].links,
                  [splitedRegex[i][j]]: states[states.length-1].id
                } 
              }
            } else {
              console.log("Error: No puede haber dos '*' seguidos");
              break a;
            }
            isClosure = false
            continue;
          }
          if(splitedRegex[i][j] !== '*') {
            if(j !== 0) {
              let links = j == splitedRegex[i].length-1?
                {[splitedRegex[i][j]]: "final"}:
                {[splitedRegex[i][j]]: states[states.length-1].id}
              states.push(this.createState(""+i+stateNumber, links))
            } else {
              if(initialState.links[splitedRegex[i][j]]) {
                initialState.links[splitedRegex[i][j]] += ','+states[states.length-1].id
              } else {
                initialState.links = {
                  ...initialState.links,
                  [splitedRegex[i][j]]: states[states.length-1].id
                }
              }
            }
            stateNumber++;
          } else {
            isClosure = true;
          }
        }
      }
      states.push(initialState);
      let AFN = {};
      states.map(s => {if(!AFN[s.id]) AFN[s.id] = s.links});
      console.log(states);
      return AFN;
    },
    createState(_id,_links) {
      let state = {
        id: _id,
        links: _links
      }
      return state
    },
    checkForPath(AFN, initState, substring) {
      console.log(AFN);
      console.log(initState);
      console.log(substring);
    },
    validateAFN(AFN, string) {
      console.log(AFN)
      let currentState = 'init';
      let i = 0;
      let done, valid = false;
      while(i < string.length && !done) {
        if(AFN[currentState][string[i]]) {
          if(AFN[currentState][string[i]].includes(",")) {
            let possibleStates = AFN[currentState][string[i]].split(",");
            possibleStates.forEach(s => {
              this.checkForPath(AFN, s, s.substring(i, string.length));
            })
          } else {
            currentState = AFN[currentState][string[i]];
            valid = currentState == 'final';
          }
        } else {
          console.log("No Match");
          done = true;
          valid = false;
        }
        i++
      }
      console.log(valid);
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