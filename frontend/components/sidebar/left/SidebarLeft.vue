<template>
  <aside
    @mouseover="collapseSidebar(false)"
    @focus="collapseSidebar(false)"
    @mouseleave="collapseSidebar(true)"
    @focusout="
      collapseSidebar(true);
      handleFocusOut($event);
    "
    ref="sidebarWrapper"
    role="menu"
    tabindex="0"
    class="elem-shadow-sm focus-brand absolute z-10 hidden h-full flex-col border-r border-light-section-div bg-light-layer-1 transition-all duration-500 dark:border-dark-section-div dark:bg-dark-layer-1 md:flex"
    :class="{
      'w-56': !sidebar.collapsed || sidebar.collapsedSwitch == false,
      'w-16': sidebar.collapsed && sidebar.collapsedSwitch == true,
      'w-60':
        (!sidebar.collapsed || sidebar.collapsedSwitch == false) &&
        sidebarContentScrollable,
      'w-20':
        sidebar.collapsed &&
        sidebar.collapsedSwitch == true &&
        sidebarContentScrollable,
    }"
  >
    <SidebarLeftHeader @toggle-pressed="setSidebarContentScrollable()" />
    <div
      ref="content"
      class="h-full overflow-x-hidden"
      :class="{
        'overflow-y-auto':
          !sidebar.collapsed || sidebar.collapsedSwitch == false,
      }"
    >
      <SearchBar class="mt-1" :location="SearchBarLocation.SIDEBAR" />
      <SidebarLeftMainSectionSelectors class="mt-2" />
      <SidebarLeftIndex
        v-if="
          sidebarType === SidebarType.ORGANIZATION_PAGE ||
          sidebarType === SidebarType.EVENT_PAGE
        "
        class="my-3"
        :name="placeholderName ? placeholderName : 'Name'"
        :sidebarType="sidebarType"
        :logoUrl="placeholderLogo"
      />
      <SidebarLeftFilters
        v-else
        :class="{
          'mx-3 py-4': !sidebar.collapsed || sidebar.collapsedSwitch == false,
          'mx-2 py-3': sidebar.collapsed && sidebar.collapsedSwitch == true,
        }"
        :filters="getFiltersByPageType"
      />
    </div>
    <SidebarLeftFooter :sidebarContentScrollable="sidebarContentScrollable" />
  </aside>
</template>

<script setup lang="ts">
import type { Filters } from "~/types/filters";
import { SearchBarLocation } from "~/types/location";
import { SidebarType } from "~/types/sidebar-type";
import {
  currentRoutePathIncludes,
  isCurrentRoutePathSubpageOf,
} from "~/utils/routeUtils";

defineProps<{
  name?: string;
}>();

const sidebar = useSidebar();
const route = useRoute();
const { currentRoute } = useRouter();
const routeName = currentRoute.value.name;
let routeToCheck = routeName;
if (routeToCheck) {
  routeToCheck = routeToCheck.toString();
} else {
  routeToCheck = "";
}

const isOrgPage = isCurrentRoutePathSubpageOf("organizations", routeToCheck);
const isEventPage = isCurrentRoutePathSubpageOf("events", routeToCheck);

const pathToSidebarTypeMap = [
  { path: "search", type: SidebarType.SEARCH },
  { path: "home", type: SidebarType.HOME },
  {
    path: "organizations",
    type: isOrgPage
      ? SidebarType.ORGANIZATION_PAGE
      : SidebarType.FILTER_ORGANIZATIONS,
  },
  {
    path: "events",
    type: isEventPage ? SidebarType.EVENT_PAGE : SidebarType.FILTER_EVENTS,
  },
];

const sidebarType =
  pathToSidebarTypeMap.find((item) =>
    currentRoutePathIncludes(item.path, routeToCheck)
  )?.type || SidebarType.MISC;

// TODO: Use real name of organization / event when available from backend.
const placeholderName = route.path.split("/").at(-2)?.replaceAll("-", " ");
const placeholderLogo = "/images/tech-from-below.svg";

const filters = {
  daysAhead: {
    sidebarType: [SidebarType.FILTER_EVENTS],
    title: "Days ahead",
    name: "daysAhead",
    type: "radio",
    style: "btn",
    allowCustomValue: true,
    items: [
      {
        label: "1",
        value: "1",
      },
      {
        label: "7",
        value: "7",
      },
      {
        label: "30",
        value: "30",
      },
    ],
  },
  eventType: {
    sidebarType: [SidebarType.FILTER_EVENTS],
    title: "Event type",
    name: "eventType",
    type: "checkbox",
    style: "btn",
    items: [
      {
        label: "Learn",
        value: "learn",
        customColor: "learn-blue",
      },
      {
        label: "Action",
        value: "action",
        customColor: "action-red",
      },
    ],
  },
  locationType: {
    sidebarType: [SidebarType.FILTER_EVENTS],
    title: "Location",
    name: "locationType",
    type: "checkbox",
    style: "btn",
    searchInput: true,
    items: [
      {
        label: "In person",
        value: "in-person",
      },
      {
        label: "Online",
        value: "online",
      },
    ],
  },
  eventLocationSearch: {
    sidebarType: [SidebarType.FILTER_EVENTS],
    title: "",
    name: "eventLocationSearch",
    type: "search",
    placeholder: "components.sidebar-left.location-search-placeholder",
  },
  locationSearch: {
    sidebarType: [SidebarType.FILTER_ORGANIZATIONS, SidebarType.SEARCH],
    title: "Location",
    name: "locationSearch",
    type: "search",
    placeholder: "components.sidebar-left.location-search-placeholder",
  },
  organizationSearch: {
    sidebarType: [SidebarType.FILTER_EVENTS],
    title: "Organization",
    name: "organizationSearch",
    type: "search",
    placeholder: "components.sidebar-left.orgs-search-placeholder",
  },
  topic: {
    sidebarType: [
      SidebarType.FILTER_EVENTS,
      SidebarType.FILTER_ORGANIZATIONS,
      SidebarType.FILTER_RESOURCES,
      SidebarType.SEARCH,
    ],
    title: "Topic",
    type: "checkbox",
    name: "topic",
    style: "simple",
    expandable: true,
    items: [
      {
        label: "Environment",
        value: "environment",
      },
      {
        label: "Housing",
        value: "housing",
      },
      {
        label: "Refugees",
        value: "refugees",
      },
      {
        label: "LGBTQIA+",
        value: "lgbtqia+",
      },
      {
        label: "Racial Justice",
        value: "racial justice",
      },
      {
        label: "Women's Rights",
        value: "women's rights",
      },
      {
        label: "Children's Rights",
        value: "children's rights",
      },
      {
        label: "Elder Rights",
        value: "elder rights",
      },
      {
        label: "Animal Rights",
        value: "animal rights",
      },
      {
        label: "Labor Rights",
        value: "labor rights",
      },
      {
        label: "Education",
        value: "education",
      },
      {
        label: "Democracy",
        value: "democracy",
      },
      {
        label: "Health",
        value: "health",
      },
      {
        label: "Privacy",
        value: "privacy",
      },
      {
        label: "Peace",
        value: "peace",
      },
      {
        label: "Nutrition",
        value: "nutrition",
      },
      {
        label: "Accessibility",
        value: "accessibility",
      },
      {
        label: "Transparency",
        value: "transparency",
      },
      {
        label: "Expression",
        value: "expression",
      },
      {
        label: "Emergency Relief",
        value: "emergency relief",
      },
      {
        label: "Infrastructure",
        value: "infrastructure",
      },
    ],
  },
};

const getFiltersByPageType = computed<Filters>(() => {
  const filteredFilters: Filters = {};
  for (const filter in filters) {
    const f = filters[filter as keyof typeof filters];
    if (!f.sidebarType.includes(sidebarType)) {
      delete filteredFilters[filter as keyof typeof filters];
    }
  }

  return filteredFilters;
});

const content = ref();

const sidebarContentScrollable = useState<boolean>(
  "sidebarContentScrollable",
  () => false
);

function setSidebarContentScrollable(): void {
  setTimeout(() => {
    sidebarContentScrollable.value =
      content.value.scrollHeight > content.value.clientHeight ? true : false;
  }, 50);
}

const sidebarWrapper = ref<HTMLElement | null>(null);

function collapseSidebar(collapse: boolean): void {
  sidebar.collapsed = collapse;
  setSidebarContentScrollable();
}

function handleFocusOut(event: FocusEvent) {
  const focusedElement = event.relatedTarget as HTMLElement;
  if (sidebarWrapper.value && sidebarWrapper.value.contains(focusedElement)) {
    collapseSidebar(false);
  } else {
    collapseSidebar(true);
  }
}

onMounted(() => {
  window.addEventListener("resize", setSidebarContentScrollable);
  setSidebarContentScrollable();
});

onUnmounted(() => {
  window.removeEventListener("resize", setSidebarContentScrollable);
});
</script>
