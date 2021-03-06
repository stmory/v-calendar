<template>
  <v-calendar
    :attributes='attributes'
    is-double-paned>
    <!--=========POPOVER HEADER SLOT=========-->
    <div
      slot='day-popover-header'
      slot-scope='{ day }'
      class='day-popover-header'>
      {{ getPopoverHeaderLabel(day) }}
    </div>
    <!--============HOW TO USE ROW SLOTS===========-->
    <!--
      STEP 1: Insert element with a unique slot name ('todo-row' in this example). Make sure slot-scope is assigned, even if not used.      
      STEP 2: In Javascript, assign that unique slot name to the 'slot' property of the attribute's popover object
    -->
    <!--===============TODO ROW SLOT==============-->
    <div
      slot='todo-row'
      slot-scope='{ customData }'
      class='todo-row'>
      <!--Todo content-->
      <div class='todo-content'>
        <!--Show textbox when editing todo-->
        <input
          class='todo-input'
          v-if='customData.id === editId'
          v-model='customData.description'
          @keyup.enter='editId = 0'
          v-focus-select />
        <!--Show status/description when not editing-->
        <span
          v-else>
          <!--Completed checkbox-->
          <input
            type='checkbox'
            v-model='customData.isComplete' />
          <!--Description-->
          <span
            :class='[
              "todo-description",
              { "complete": customData.isComplete }]'
            @click='toggleTodoComplete(customData)'>
            {{ customData.description }}
          </span>
        </span>
      </div>
      <!--Edit/Done buttons-->
      <a @click.prevent='toggleTodoEdit(customData)'>
        <!--Edit button-->
        <b-icon
          v-if='editId !== customData.id'
          icon='pencil'
          type='is-info'
          size='is-small'>
        </b-icon>
        <!--Done button-->
        <b-icon v-else
          icon='check'
          type='is-success'
          size='is-small'>
        </b-icon>
      </a>
      <!--Delete button-->
      <a
        @click.prevent='deleteTodo(customData)'
        v-if='!editId || editId !== customData.id'
        class='delete-todo'>
        <b-icon
          icon='trash'
          type='is-danger'
          size='is-small'>
        </b-icon>
      </a>
    </div>
    <!--================ADD TODO ROW SLOT===============-->
    <div
      slot='add-todo'
      slot-scope='{ day }'
      class='add-todo'>
      <a @click='addTodo(day)'>
        + Add Todo
      </a>
    </div>
  </v-calendar>
</template>

<script>
import { getExampleMonthComps } from '@/utils/helpers';

const { thisMonth, thisMonthYear } = getExampleMonthComps();

const color = '#ff8080';
const todos = [
  {
    id: 1,
    description: 'Take Noah to basketball practice.',
    isComplete: false,
    dates: new Date(thisMonthYear, thisMonth, 1),
  },
  {
    id: 2,
    description: 'Get some milks.',
    isComplete: false,
    dates: new Date(thisMonthYear, thisMonth, 5),
  },
  {
    id: 3,
    description: 'Pay the utility bill.',
    isComplete: false,
    dates: new Date(thisMonthYear, thisMonth, 19),
  },
  {
    id: 4,
    description: 'Pick up clothes from the cleaners.',
    isComplete: false,
    dates: new Date(thisMonthYear, thisMonth, 19),
  },
  {
    id: 5,
    description: 'Lunch with Leo.',
    isComplete: false,
    dates: new Date(thisMonthYear, thisMonth, 22),
  },
];

export default {
  data() {
    return {
      incId: todos.length,
      editId: 0,
      todos,
    };
  },
  computed: {
    attributes() {
      return [
        // Today attribute
        {
          contentStyle: {
            fontWeight: '700',
            color: '#66b3cc',
          },
          dates: new Date(),
        },
        // Weekends are for resting...
        {
          contentStyle: {
            opacity: 0.5,
            pointerEvents: 'none',
          },
          dates: { weekends: true },
          popover: {
            visibility: 'hidden', // Hide popovers too
          },
          order: 100, // High order gives attribute high priority
        },
        // Todo attributes
        ...this.todos.map(todo => ({
          key: todo.id,
          dates: todo.dates,
          customData: todo,
          order: todo.id,
          dot: {
            backgroundColor: color,
            opacity: todo.isComplete ? 0.3 : 1,
          },
          popover: {
            slot: 'todo-row', // Matches slot from above
            visibility: 'focus',
          },
          contentStyle: ({ day }) => day.day === 5 && { color: 'red' },
        })),
        // 'Add todo' attribute
        {
          contentStyle: ({ day, isHovered, isFocused }) =>
            (isHovered || isFocused) && {
              backgroundColor: 'rgba(0, 0, 0, 0.1)',
              cursor: 'pointer',
            },
          dates: {},
          popover: {
            slot: 'add-todo',
            visibility: 'focus',
            hideIndicator: true,
          },
        },
      ];
    },
  },
  methods: {
    getPopoverHeaderLabel(day) {
      const options = {
        weekday: 'short',
        year: 'numeric',
        month: 'short',
        day: 'numeric',
      };
      return day.date.toLocaleDateString(
        window.navigator.userLanguage || window.navigator.language,
        options,
      );
    },
    addTodo(day) {
      this.editId = ++this.incId;
      this.todos.push({
        id: this.editId,
        description: 'New todo',
        isComplete: false,
        dates: day.date,
      });
    },
    toggleTodoComplete(todo) {
      todo.isComplete = !todo.isComplete;
    },
    toggleTodoEdit(todo) {
      this.editId = this.editId === todo.id ? 0 : todo.id;
    },
    deleteTodo(todo) {
      this.todos = this.todos.filter(t => t !== todo);
    },
  },
  directives: {
    focusSelect: {
      inserted(el) {
        el.focus();
        el.select();
      },
    },
  },
};
</script>

<style lang='sass' scoped>

.day-popover-header
  text-align: center
  padding-bottom: 3px
  border-bottom: 1px solid #dadada
  margin: 0 0 3px 0
  opacity: 0.7

.todo-row
  display: flex
  flex-wrap: none
  width: 100%

.todo-content
  flex-grow: 1
  flex-basis: 0
  margin-right: 10px
  min-width: 80px
  .todo-input
    width: 100%
    min-width: 200px
  .todo-description
    cursor: pointer
    transition: all 0.1s ease-in-out
    margin-left: 3px
    &:hover
      opacity: 0.5
    &.complete
      text-decoration: line-through

.add-todo
  font-size: 0.8rem
  text-align: center
  width: 100%

.delete-todo
  margin-left: 3px

</style>
