<template>
<v-date-picker
  mode='single'
  v-model='selectedValue'
  is-expanded>
  <b-field :type='inputState.type' slot-scope='props'>
    <b-input
      type='text'
      icon='calendar'
      :value='props.inputValue'
      :placeholder='inputState.message'
      @change.native='props.updateValue($event.target.value)'
      expanded>
    </b-input>
    <p
      class='control'
      v-if='selectedValue'>
      <a
        :class='["button", inputState.type]'
        @click='selectedValue = null'>
        Clear
      </a>
    </p>
  </b-field>
</v-date-picker>
</template>

<script>
export default {
  data() {
    return {
      selectedValue: null,
    };
  },
  computed: {
    inputState() {
      if (!this.selectedValue) {
        return {
          type: 'is-danger',
          message: 'Date required.',
        };
      }
      return {
        type: 'is-primary',
        message: '',
      };
    },
  },
};
</script>
