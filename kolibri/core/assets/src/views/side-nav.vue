<template>

  <div @keydown.esc="toggleNav" ref="sideNav">
    <transition name="side-nav">
      <div
        v-show="navShown"
        class="side-nav"
        :style="{ width: `${width}px` }"
      >
        <div
          class="side-nav-header"
          :style="{
            height: headerHeight + 'px',
            width: `${width}px`, paddingTop: mobile ? '4px' : '8px'
          }"
        >
          <ui-icon-button
            ref="toggleButton"
            :aria-label="$tr('closeNav')"
            type="secondary"
            color="white"
            size="large"
            @click="toggleNav"
          >
            <mat-svg
              name="close"
              category="navigation"
              class="side-nav-header-close"
            />
          </ui-icon-button>
          <span class="side-nav-header-name">{{ $tr('kolibri') }}</span>
        </div>

        <div
          class="side-nav-scrollable-area"
          :style="{ top: `${headerHeight}px`, width: `${width}px` }"
        >
          <core-menu
            class="side-nav-scrollable-area-menu"
            role="navigation"
            :hasIcons="true"
            :aria-label="$tr('navigationLabel')"
          >
            <template slot="options">
              <component v-for="component in menuOptions" :is="component" :key="component.name" />
              <divider />
            </template>
          </core-menu>

          <div class="side-nav-scrollable-area-footer">
            <logo class="side-nav-scrollable-area-footer-logo" />
            <div class="side-nav-scrollable-area-footer-info">
              <p>{{ footerMsg }}</p>
              <!-- Not translated -->
              <p>© 2017 Learning Equality</p>
            </div>
          </div>
        </div>

      </div>
    </transition>

    <div
      v-show="navShown"
      class="side-nav-backdrop"
      @click="toggleNav"
    >
    </div>
  </div>

</template>


<script>

  import { UserKinds, NavComponentSections } from 'kolibri.coreVue.vuex.constants';
  import responsiveWindow from 'kolibri.coreVue.mixins.responsiveWindow';
  import responsiveElement from 'kolibri.coreVue.mixins.responsiveElement';
  import coreMenu from 'kolibri.coreVue.components.coreMenu';
  import coreMenuOption from 'kolibri.coreVue.components.coreMenuOption';
  import uiIconButton from 'keen-ui/src/UiIconButton';
  import logo from 'kolibri.coreVue.components.logo';
  import navComponents from 'kolibri.utils.navComponents';
  import navComponentsMixin from '../mixins/nav-components';
  import logout from './logout-side-nav-entry';
  import divider from './side-nav-divider';

  // Explicit ordered list of roles for nav item sorting
  const navComponentRoleOrder = [
    UserKinds.ANONYMOUS,
    UserKinds.LEARNER,
    UserKinds.COACH,
    UserKinds.ADMIN,
    UserKinds.CAN_MANAGE_CONTENT,
    UserKinds.SUPERUSER,
  ];

  export default {
    name: 'sideNav',
    components: {
      coreMenu,
      uiIconButton,
      logo,
      coreMenuOption,
      divider,
    },
    mixins: [responsiveWindow, responsiveElement, navComponentsMixin],
    $trs: {
      kolibri: 'Kolibri',
      navigationLabel: 'Main user navigation',
      closeNav: 'Close navigation',
      poweredBy: 'Kolibri {version}',
    },
    props: {
      navShown: {
        type: Boolean,
        required: true,
      },
      headerHeight: {
        type: Number,
        required: true,
      },
      width: {
        type: Number,
        required: true,
      },
    },
    data() {
      return {
        previouslyFocusedElement: null,
      };
    },
    computed: {
      mobile() {
        return this.windowSize.breakpoint < 2;
      },
      footerMsg() {
        return this.$tr('poweredBy', { version: __version });
      },
      menuOptions() {
        const topComponents = navComponents
          .filter(component => component.section !== NavComponentSections.ACCOUNT)
          .sort(this.compareMenuComponents);
        const accountComponents = navComponents
          .filter(component => component.section === NavComponentSections.ACCOUNT)
          .sort(this.compareMenuComponents);
        return [...topComponents, divider, ...accountComponents, logout].filter(this.filterByRole);
      },
    },
    watch: {
      navShown(isShown) {
        this.$nextTick(() => {
          if (isShown) {
            window.addEventListener('focus', this.containFocus, true);
            this.previouslyFocusedElement = document.activeElement;
            this.$refs.toggleButton.$el.focus();
          } else {
            window.removeEventListener('focus', this.containFocus, true);
            this.previouslyFocusedElement.focus();
          }
        });
      },
    },
    methods: {
      toggleNav() {
        this.$emit('toggleSideNav');
      },
      compareMenuComponents(navComponentA, navComponentB) {
        // Compare menu items to allow sorting by the following priority:
        // Sort by role
        // Nav items with no roles will be placed first
        // as index will be -1
        if (navComponentA.role !== navComponentB.role) {
          return (
            navComponentRoleOrder.indexOf(navComponentA.role) -
            navComponentRoleOrder.indexOf(navComponentB.role)
          );
        }
        // Next sort by priority
        if (navComponentA.priority !== navComponentB.priority) {
          return navComponentA.priority - navComponentB.priority;
        }
        // Still no difference?
        // There is no difference!
        return 0;
      },
      containFocus(event) {
        if (event.target === window) {
          return event;
        }
        if (!this.$refs.sideNav.contains(event.target)) {
          this.$refs.toggleButton.$el.focus();
        }
        return event;
      },
    },
    vuex: {
      getters: {
        session: state => state.core.session,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  // matches angular material's spec
  $side-nav-box-shadow =  0 2px 4px -1px rgba(0, 0, 0, 0.2),
                          0 4px 5px 0 rgba(0, 0, 0, 0.14),
                          0 1px 10px 0 rgba(0, 0, 0, 0.12)

  // matches keen-ui toolbar's spec
  $side-nav-header-box-shadow = 0 0 2px rgba(black, 0.12), 0 2px 2px rgba(black, 0.2)

  .side-nav
    position: fixed
    top: 0
    bottom: 0
    z-index: 16
    color: $core-text-default
    background: $core-bg-light
    box-shadow: $side-nav-box-shadow

  .side-nav-enter
    transform: translate3D(-100%, 0, 0)

  .side-nav-enter-active
    transition: all 0.2s ease-in-out

  .side-nav-enter-to
    transform: translate3D(0, 0, 0)

  .side-nav-leave
    transform: translate3D(0, 0, 0)

  .side-nav-leave-active
    transition: all 0.2s ease-in-out

  .side-nav-leave-to
    transform: translate3D(-100%, 0, 0)

  .side-nav-header
    position: fixed
    top: 0
    left: 0
    z-index: 17
    font-size: 14px
    text-transform: uppercase
    background-color: $core-text-default
    box-shadow: $side-nav-header-box-shadow

  .side-nav-header-close
    fill: white

  .side-nav-header-name
    margin-left: 8px
    vertical-align: middle
    color: $core-bg-light
    font-weight: bold
    font-size: 18px

  .side-nav-scrollable-area
    position: fixed
    left: 0
    bottom: 0
    overflow: auto

  .side-nav-scrollable-area-menu
    background: $core-bg-light

  .side-nav-scrollable-area-footer
    color: $core-text-annotation
    padding: 16px

  .side-nav-scrollable-area-footer-logo
    height: 77px
    max-width: 100%

  .side-nav-scrollable-area-footer-info
    margin-top: 8px
    font-size: 12px
    line-height: 16px
    p
      margin: 0

  .side-nav-backdrop
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    background: rgba(0, 0, 0, 0.7)
    transition: opacity 0.3s ease
    background-attachment: fixed
    z-index: 15


  /* keen menu */
  >>>.ui-menu
    max-height: none
    padding: 0
    background: $core-bg-light
    border: none

  >>>.ui-menu-option
    &:not(.is-divider)
      padding-top: 4px
      padding-bottom: 4px

      .ui-menu-option-text
        overflow: visible
        white-space: normal
        color: $core-text-default
        font-size: 14px

      .ui-menu-option-icon
        color: $core-text-default
        font-size: 1.2em

      &.is-active
        .ui-menu-option-text
          color: $core-accent-color
          font-weight: bold
          opacity: 1

        .ui-menu-option-icon
          color: $core-accent-color

    &.is-divider
      margin-top: 0
      margin-bottom: 0

</style>
