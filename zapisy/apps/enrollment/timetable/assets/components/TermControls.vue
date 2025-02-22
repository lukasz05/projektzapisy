<script lang="ts">
// This component presents a single group term in a week. It extends upon
// TermComponent by adding control buttons that allow to enqueue/dequeue to
// the group and to pin it.
import Component from "vue-class-component";
import Vue from "vue";
import TermComponent from "./Term.vue";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

import { Term, Group } from "../models";

const TermControlsProps = Vue.extend({
  props: {
    term: Term,
  },
});

@Component({
  components: {
    Term: TermComponent,
    FontAwesomeIcon,
  },
})
export default class TermControlsComponent extends TermControlsProps {
  controlsVisible: boolean = false;

  get group(): Group {
    return this.term.group;
  }

  // Determines if a new enrollment record can be created into the group.
  get canEnqueue(): boolean {
    if (this.term.group.isEnrolled) return false;
    if (this.term.group.isEnqueued) return false;
    return this.term.group.canEnqueue;
  }

  // Determines if an enrollment record exists and can be removed.
  get canDequeue(): boolean {
    if (!this.term.group.canDequeue) return false;
    if (this.term.group.isEnrolled) return true;
    if (this.term.group.isEnqueued) return true;
    return false;
  }

  pin() {
    this.$store.dispatch("groups/pin", this.term.group);
  }

  unpin() {
    this.$store.dispatch("groups/unpin", this.term.group);
  }

  enqueue() {
    const confirmMessage = [
      "Czy na pewno chcesz stanąć w kolejce do tej grupy?\n\n",
      "Gdy tylko w grupie będzie wolne miejsce (być może natychmiast), ",
      "zostanie dokonana próba wciągnięcia do niej studentów z kolejki. Jeśli ",
      "w momencie wciągania do grupy student nie spełnia warunków zapisu ",
      "(np. przekracza limit ECTS), jego rekord zostaje usunięty.",
    ].join("");

    if (confirm(confirmMessage)) {
      this.$store.dispatch("groups/enqueue", this.term.group);
    }
  }

  dequeue() {
    const confirmMessage = "Czy na pewno chcesz opuścić tę grupę/kolejkę?";
    if (confirm(confirmMessage)) {
      this.$store.dispatch("groups/dequeue", this.term.group);
    }
  }

  showControls() {
    this.controlsVisible = true;
    window.addEventListener("touchend", this.hideControls);
  }

  // Hides controls popup whenever there is a "mouseout" event on the Term
  // component or any touch event outside of it.
  hideControls(event: Event) {
    // Do not hide controls on touch events inside of the Term.
    if (
      event.type === "touchend" &&
      ((this.$refs.term as Vue).$refs.root as Element).contains(
        event.target as Node
      )
    ) {
      return;
    }
    this.controlsVisible = false;
    window.removeEventListener("touchend", this.hideControls);
  }
}
</script>

<template>
  <Term
    :term="term"
    @mouseover.native="showControls"
    @mouseout.native="hideControls"
    ref="term"
  >
    <transition name="fade">
      <div class="controls" v-if="controlsVisible">
        <span
          v-if="group.isPinned"
          class="unpin"
          title="Odepnij grupę od planu."
          @click="unpin()"
        >
          <font-awesome-icon
            icon="thumbtack"
            :transform="{ rotate: 45 }"
            fixed-width
          />
        </span>
        <span
          v-else
          class="pin"
          title="Przypnij grupę do planu."
          @click="pin()"
        >
          <font-awesome-icon icon="thumbtack" rotation="90" fixed-width />
        </span>

        <span
          v-if="canEnqueue"
          class="enqueue"
          title="Zapisz do grupy/kolejki."
          @click="enqueue()"
        >
          <font-awesome-icon icon="pencil-alt" fixed-width />
        </span>
        <span
          v-if="canDequeue"
          class="dequeue"
          title="Wypisz z grupy/kolejki."
          @click="dequeue()"
        >
          <font-awesome-icon icon="ban" fixed-width />
        </span>
        <span
          v-if="term.group.autoEnrollment"
          class="auto-enrollment"
          title="Grupa z auto-zapisem."
        >
          <font-awesome-icon icon="car-side" fixed-width />
        </span>
      </div>
    </transition>
  </Term>
</template>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.controls {
  background: white;

  position: absolute;
  top: -1px;
  left: -1px;

  cursor: default;
  border: 1px solid #666666;
  border-radius: 4px 0;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.5);
  z-index: 30;

  display: flex;
  flex-direction: column;
}

.controls span {
  display: block;
  padding: 3px;
  width: 22px;
  height: 22px;
  font-size: 14px;
  cursor: pointer;
}

// For devices with large screens where days are displayed one below another.
// Bootstrap's convention: https://getbootstrap.com/docs/4.5/layout/overview/#containers
@media (max-width: 992px) {
  .controls {
    position: absolute;
    top: -63px;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 4px;

    min-width: 60px;
    flex-direction: row;

    // Tooltip arrow.
    &:before {
      content: "";
      display: block;
      position: absolute;
      left: calc(50% - 10px);
      top: 100%;
      border: 10px solid transparent;
      border-top-color: black;
    }
    &:after {
      content: "";
      display: block;
      position: absolute;
      left: calc(50% + 1px - 10px);
      top: 100%;
      border: 9px solid transparent;
      border-top-color: white;
    }
  }

  .controls span {
    display: inline-block;
    font-size: 40px;
    padding: 5px;
    width: 60px;
    height: 50px;
  }
}
</style>
