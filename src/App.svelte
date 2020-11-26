<script>
  if (navigator.requestMIDIAccess) {
    console.log("This browser supports WebMIDI!");
  } else {
    console.log("WebMIDI is not supported in this browser.");
  }
  /* navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure); */

  /* function onMIDISuccess(midiAccess) { */
  /*   console.log(midiAccess); */

  /*   var inputs = midiAccess.inputs; */
  /*   var outputs = midiAccess.outputs; */
  /* } */

  /* function onMIDIFailure() { */
  /*   console.log("Could not access your MIDI devices."); */
  /* } */
</script>

<style>
</style>

<main>
  <h2>Midi Keyboard App</h2>
</main>
