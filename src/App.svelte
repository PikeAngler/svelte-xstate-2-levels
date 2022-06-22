<script lang="ts">
  import { createMachine, interpret, matchesState } from 'xstate';

  const onOffMachine = createMachine({
    initial: 'off',
    states: {
      off: { on: { CLICK: 'on'}},
      on: {
        on: {   
          CLICK: {
            target: 'off',
            in: 'on.green'
          },
        },
        initial: 'red',
        states: {
          red: { on: { STEP: 'yellow' } },
          yellow: { on: { STEP: 'green' } },
          green: { on: { STEP: 'red' } }
        }
      }
    }
  });

  let stateValue;
  let isOn;

  const onOffInterpreter = interpret(onOffMachine);
  onOffInterpreter.init();

  onOffInterpreter.onTransition(
    (state) => {
      stateValue = state.value
      isOn = matchesState('on', stateValue)
    }
  );

  $: { 
    console.log(stateValue);
    console.log("is red:", matchesState('on.red', stateValue));
  }

</script>

<style>
  h1, h2 {
    font-family: Lato;
  }

  * {
    font-size: 18px;
  }

  #onOffBtn {
    display: inline-block;
    padding: 10px;
    color: white;
    border-radius: 10px;
    cursor: pointer;
    border: 2px solid transparent;  
  }

  #onOffBtn:hover {
    border: 2px solid grey;
  }

  #lightsContainer {
    display: flex;  
    margin: 15px;
  }

  .lights {
    display: inline-block;
    margin: 10px;
    padding: 5px;
    border-radius: 20px;
    background-color: black;
  }
  .lights-div {
    margin: 3px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background-color: grey;
  }
  .flex {
    display: flex;
  }
  .yellow {
    background-color: yellow;  
  }
  .green {
    background-color: green;
  }
  .red {
    background-color:red;
  }
</style>

<div id="app">
  <div 
    id="onOffBtn"
    on:click={() => onOffInterpreter.send('CLICK')}
    class:red={!isOn}
    class:green={isOn}>
    { isOn ? 'On' : 'Off'}
  </div>
  <div 
    id="lightsContainer"
    class:flex={isOn}>
    <button 
      id="stepBtn"
      on:click={() => {onOffInterpreter.send('STEP')}}>
      Do Step
    </button>
    <div class="lights">
      <div 
        class="lights-div"
        class:red={matchesState('on.red', stateValue)}>
      </div>
      <div 
        class="lights-div"
        class:yellow={matchesState('on.yellow', stateValue)}>
      </div>
      <div 
        class="lights-div"
        class:green={matchesState('on.green', stateValue)}>
      </div>
    </div>
  </div>
  Can only be turned off when green.
</div>
