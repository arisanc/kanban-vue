<template>
  <div class="drag-container">
    <ul class="drag-list left">
      <div
        v-for="(kanban, index) in kanbanList"
        :key="kanban.label"
        :class="index % limit === 0 ? 'kb-href' : ''"
      >
        <li
          class="drag-column"
          :class="{ ['drag-column-' + kanban.label]: true }"
          :style="{ width: 80 / limit + 'vw', background: config.kanbanBg }"
        >
          <span
            class="drag-column-header"
            :style="{
              background: kanban.background,
              color: kanban.fontColor || kanban.color,
            }"
          >
            <h2>
              {{ kanban.label }}
              <a>({{ kanban.tasks.length }})</a>
              <img
                class="kb-icon kb-icon-title-edit"
                @click.prevent="editKanban(kanban)"
                src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAxOS4xLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCA2NCA2NCIgZW5hYmxlLWJhY2tncm91bmQ9Im5ldyAwIDAgNjQgNjQiIHhtbDpzcGFjZT0icHJlc2VydmUiPg0KPGcgaWQ9IkdyYXBoaWMtZGVzaWduXzJfIj4NCgk8cGF0aCBkPSJNNDkuNzU3NDAwNSwzLjY0MTY3NWMtMC4yMTc0OTg4LTAuMTUyMDk5OC0wLjQ4NTU5OTUtMC4yMTE0OTk5LTAuNzQ2ODk4Ny0wLjE2MzY5OTkNCgkJYy0wLjI2MDI5OTcsMC4wNDY3LTAuNDkxNDAxNywwLjE5NDktMC42NDI0MDI2LDAuNDEyMjk5OUwyNS4zNTE4MDA5LDM2Ljk1MzM3NjgNCgkJYy0wLjA4ODgwMDQsMC4xMjY2OTc1LTAuMTQ2MzAxMywwLjI3Mjg5OTYtMC4xNjg3MDEyLDAuNDI2OTk4MWwtMS41MTc5OTk2LDEwLjQzMTgwMDgNCgkJYy0wLjA1NDU5OTgsMC4zNzMzOTc4LDAuMTA3Mjk5OCwwLjc0Njc5OTUsMC40MTczMDEyLDAuOTYyMjk5M2MwLjE3MDU5OSwwLjExODkwMDMsMC4zNzA0OTg3LDAuMTc5NDAxNCwwLjU3MDI5OTEsMC4xNzk0MDE0DQoJCWMwLjE2Mzc5OTMsMCwwLjMyODU5OTktMC4wNDAwMDA5LDAuNDc3ODAwNC0wLjEyMTkwMjVsOS4yNTYwMDA1LTUuMDQ0MzAwMQ0KCQljMC4xMzY0OTc1LTAuMDc0MTAwNSwwLjI1MzQ5ODEtMC4xNzgzOTgxLDAuMzQxMTk4LTAuMzA2MTk4MUw1Ny43NDMsMTAuNDE4NDc1Mg0KCQljMC4zMTQ5OTg2LTAuNDUyNDAwMiwwLjIwMzgwMDItMS4wNzQzOTk5LTAuMjQ4Ni0xLjM4OTMwMDNMNDkuNzU3NDAwNSwzLjY0MTY3NXogTTMzLjIyNDM5OTYsNDIuMTQ3NzczN2wtNy4yOTY0OTkzLDMuOTc1Nzk5Ng0KCQlsMS4xOTczLTguMjIyNTk5bDIyLjMxMDQtMzIuMDQ5OTk5Mmw2LjA5OTIwMTIsNC4yNDU4TDMzLjIyNDM5OTYsNDIuMTQ3NzczN3oiLz4NCgk8cGF0aCBkPSJNNTYuMjE3MzAwNCwyMy42MjQ5NzUyYy0wLjU1MTg5OSwwLTAuOTk4NDAxNiwwLjQ0NjUwMDgtMC45OTg0MDE2LDAuOTk4Mzk5N3YzMy40OTU4DQoJCWMwLDIuMTQxOTk4My0xLjc0MjE5ODksMy44ODQyMDEtMy44ODQwOTgxLDMuODg0MjAxSDkuMTg2NDAwNGMtMi4xNDIwMDAyLDAtMy44ODQyMDAxLTEuNzQyMjAyOC0zLjg4NDIwMDEtMy44ODQyMDFWMTUuOTcwNzc0Nw0KCQljMC0yLjE0MTg5OTEsMS43NDIxOTk5LTMuODg0MTk5MSwzLjg4NDIwMDEtMy44ODQxOTkxaDI0Ljg0MzI5OTljMC41NTE3OTk4LDAsMC45OTgyOTg2LTAuNDQ2NTAwOCwwLjk5ODI5ODYtMC45OTgzMDA2DQoJCXMtMC40NDY0OTg5LTAuOTk4Mjk5Ni0wLjk5ODI5ODYtMC45OTgyOTk2SDkuMTg2NDAwNGMtMy4yNDI3MDAxLDAtNS44ODA5MDA0LDIuNjM4MTk5OC01Ljg4MDkwMDQsNS44ODA3OTkzVjU4LjExOTE3NQ0KCQljMCwzLjI0MjU5OTUsMi42MzgyLDUuODgwNzk4Myw1Ljg4MDkwMDQsNS44ODA3OTgzaDQyLjE0ODM5OTRjMy4yNDI1OTk1LDAsNS44ODA3OTgzLTIuNjM4MTk4OSw1Ljg4MDc5ODMtNS44ODA3OTgzdi0zMy40OTU4DQoJCUM1Ny4yMTU1OTkxLDI0LjA3MTQ3Niw1Ni43NjkxMDAyLDIzLjYyNDk3NTIsNTYuMjE3MzAwNCwyMy42MjQ5NzUyeiIvPg0KCTxwYXRoIGQ9Ik02MC4yNDk1OTk1LDUuNTA2Nzc0OWwtOC4wMDgwOTg2LTUuMzM4OGMtMC40NjAyMDEzLTAuMzA2LTEuMDc5Mjk5OS0wLjE4MjMtMS4zODQzOTk0LDAuMjc3DQoJCWMtMC4zMDYyMDE5LDAuNDU5MS0wLjE4MjMwMDYsMS4wNzgyLDAuMjc2Nzk4MiwxLjM4NDRsOC4wMDgyMDE2LDUuMzM4Njk5OA0KCQljMC4xNzA2MDA5LDAuMTEzMTAwMSwwLjM2MjU5ODQsMC4xNjc2OTk4LDAuNTUyNzk5MiwwLjE2NzY5OThjMC4zMjI2MDEzLDAsMC42Mzk0OTk3LTAuMTU1OTk5NywwLjgzMTYwMDItMC40NDQ1OTk2DQoJCUM2MC44MzI1OTk2LDYuNDMxOTc0OSw2MC43MDg4MDEzLDUuODEyODc1Myw2MC4yNDk1OTk1LDUuNTA2Nzc0OXoiLz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjwvc3ZnPg0K"
              />
            </h2>
          </span>

          <div class="drag-options"></div>
          <ul
            class="drag-inner-list"
            ref="list"
            :data-status="kanban.name || kanban.label"
            :data-index="index"
            :style="{ color: kanban.background }"
          >
            <li
              :class="task.go ? 'drag-item drag-item-leave' : 'drag-item'"
              v-for="(task, index) in kanban.tasks"
              :data-block-id="task.id"
              :index="index"
              :key="task.id"
              @click="editTask(task)"
              :style="{
                background: config.taskBg || kanban.color,
                color: kanban.fontColor,
                marginTop: task.top + 'px',
                marginBotton: task.bottom + 'px',
              }"
            >
              <div class="kb-img-container" v-if="task.img && task.img.length">
                <img
                  class="kb-img"
                  :src="task.img[0]"
                  :style="{
                    maxWidth: task.imgsize + '%',
                    maxHeight: task.imgsize + '%',
                  }"
                />
              </div>
              <!--:style="{cardStyle}" -->
              <div class="kb-card-title">
                {{ task.title }}
              </div>
              <div class="kb-tags" v-if="task.tags && task.tags.length">
                <span
                  v-for="(tag, i) in task.tags"
                  :key="i"
                  class="kb-tag"
                  :style="{
                    background: tagConfigs && tagConfigs[tag].bg,
                    color: tagConfigs && tagConfigs[tag].color,
                  }"
                >
                  {{ tag }}
                </span>
              </div>
              <div class="kb-card-tags"></div>
              <div
                class="kb-card-content"
                :style="{ '-webkit-line-clamp': config.contentLimit || 3 }"
                v-if="task.content"
              >
                {{ task.content }}
              </div>
              <div v-if="task.todo && task.todo.length">
                <ul class="kb-todo">
                  <li
                    class="kb-todo-content"
                    v-for="(todo, i) in task.todo"
                    :key="i"
                  >
                    {{ todo }}
                  </li>
                </ul>
              </div>

              <div class="kb-card-progress">
                <progress-bar
                  style="width: 30%"
                  :bar-color="$parent.getColor(task)"
                  :bg-color="task.bg || '#ccc'"
                  :val="'' + task.progress"
                  v-if="task.progress"
                  :text="task.progress + '%'"
                  :title="task.progress + '%'"
                  text-position="bottom"
                  bar-transition="all 2s ease"
                  size="small"
                  text-align="left"
                ></progress-bar>
                <div class="kb-card-dead-time" v-if="task.time">
                  <img
                    class="kb-card-dead-time-icon kb-icon"
                    src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/PjwhRE9DVFlQRSBzdmcgIFBVQkxJQyAnLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4nICAnaHR0cDovL3d3dy53My5vcmcvR3JhcGhpY3MvU1ZHLzEuMS9EVEQvc3ZnMTEuZHRkJz48c3ZnIGVuYWJsZS1iYWNrZ3JvdW5kPSJuZXcgMCAwIDQ4IDQ4IiBoZWlnaHQ9IjQ4cHgiIGlkPSJMYXllcl8xIiB2ZXJzaW9uPSIxLjEiIHZpZXdCb3g9IjAgMCA0OCA0OCIgd2lkdGg9IjQ4cHgiIHhtbDpzcGFjZT0icHJlc2VydmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPjxwYXRoIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTQzLjEwMiwxNC4xMTNjLTAuMzMxLTAuNTc5LTAuNjg0LTEuMTQzLTEuMDY0LTEuNjg4ICBDNDMuMjA4LDExLjcyNyw0NCwxMC40NjIsNDQsOWMwLTIuMjA5LTEuNzkxLTQtNC00Yy0xLjQ2MiwwLTIuNzI3LDAuNzkyLTMuNDI1LDEuOTYzYy0wLjU0NS0wLjM4MS0xLjEwOC0wLjczMy0xLjY4OC0xLjA2NCAgQzM1LjkzOSw0LjE2NywzNy44MjYsMyw0MCwzYzMuMzEzLDAsNiwyLjY4Nyw2LDZDNDYsMTEuMTc0LDQ0LjgzMiwxMy4wNjEsNDMuMTAyLDE0LjExM3ogTTQ0LDI1YzAsNS4xNzEtMS45NzksOS44NjgtNS4yLDEzLjQxOSAgbDQuOTE1LDQuOTE1YzAuMzgxLDAuMzgxLDAuMzgxLDAuOTk5LDAsMS4zODFjLTAuMzgyLDAuMzgxLTEsMC4zODEtMS4zODEsMEwzNy40MTksMzkuOEMzMy44NjgsNDMuMDIxLDI5LjE3MSw0NSwyNCw0NSAgcy05Ljg2OC0xLjk3OS0xMy40MTgtNS4ybC00LjkxNiw0LjkxNWMtMC4zODEsMC4zODEtMSwwLjM4MS0xLjM4MSwwYy0wLjM4MS0wLjM4Mi0wLjM4MS0xLDAtMS4zODFMOS4yLDM4LjQxOSAgQzUuOTc5LDM0Ljg2OCw0LDMwLjE3MSw0LDI1QzQsMTMuOTU0LDEyLjk1NCw1LDI0LDVTNDQsMTMuOTU0LDQ0LDI1eiBNMjQsN0MxNC4wNTksNyw2LDE1LjA1OSw2LDI1YzAsOS45NDEsOC4wNTksMTgsMTgsMTggIHMxOC04LjA1OSwxOC0xOEM0MiwxNS4wNTksMzMuOTQxLDcsMjQsN3ogTTI1LDI4Ljg1OFYzOGMwLDAuNTUzLTAuNDQ3LDEtMSwxcy0xLTAuNDQ3LTEtMXYtOS4xNDJjLTEuNzIzLTAuNDQ2LTMtMS45OTctMy0zLjg1OCAgYzAtMC43NDksMC4yMTktMS40NDEsMC41NzgtMi4wNDFsLTQuMjkzLTQuMjkyYy0wLjM4MS0wLjM4MS0wLjM4MS0xLDAtMS4zODFjMC4zODItMC4zODEsMS0wLjM4MSwxLjM4MSwwbDQuMjkzLDQuMjkyICBDMjIuNTU5LDIxLjIxOSwyMy4yNTEsMjEsMjQsMjFjMi4yMDksMCw0LDEuNzkxLDQsNEMyOCwyNi44NjEsMjYuNzIzLDI4LjQxMiwyNSwyOC44NTh6IE0yNCwyM2MtMS4xMDQsMC0yLDAuODk2LTIsMnMwLjg5NiwyLDIsMiAgczItMC44OTYsMi0yUzI1LjEwNCwyMywyNCwyM3ogTTgsNUM1Ljc5MSw1LDQsNi43OTEsNCw5YzAsMS40NjIsMC43OTMsMi43MjcsMS45NjMsMy40MjVjLTAuMzgxLDAuNTQ1LTAuNzMzLDEuMTA5LTEuMDY0LDEuNjg4ICBDMy4xNjgsMTMuMDYxLDIsMTEuMTc0LDIsOWMwLTMuMzEzLDIuNjg3LTYsNi02YzIuMTc0LDAsNC4wNjEsMS4xNjcsNS4xMTMsMi44OTljLTAuNTgsMC4zMzEtMS4xNDQsMC42ODMtMS42ODgsMS4wNjQgIEMxMC43MjcsNS43OTIsOS40NjEsNSw4LDV6IiBmaWxsLXJ1bGU9ImV2ZW5vZGQiLz48L3N2Zz4="
                  />
                  {{ task.time }}
                </div>
              </div>
              <div class="kb-card-more">
                <div class="kb-card-worker" v-if="task.worker">
                  <img
                    class="kb-icon"
                    src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/Pjxzdmcgdmlld0JveD0iMCAwIDMyIDMyIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjx0aXRsZS8+PGcgZGF0YS1uYW1lPSJMYXllciAyIiBpZD0iTGF5ZXJfMiI+PHBhdGggZD0iTTE2LDE3LjVBNy41LDcuNSwwLDEsMSwyMy41LDEwLDcuNSw3LjUsMCwwLDEsMTYsMTcuNVptMC0xNEE2LjUsNi41LDAsMSwwLDIyLjUsMTAsNi41MSw2LjUxLDAsMCwwLDE2LDMuNVoiLz48cGF0aCBkPSJNMjYsMjguNUg2YS41LjUsMCwwLDEtLjUtLjVWMjJhLjY4LjY4LDAsMCwxLDAtLjE0Yy40OC0xLjY2LDItMi44MSwzLjQtMy44Mi4zMy0uMjUuNjQtLjQ4LjkzLS43MmEuNTEuNTEsMCwwLDEsLjY4LjA1QTcuMTcsNy4xNywwLDAsMCwxNiwxOS41YTcuMTEsNy4xMSwwLDAsMCw1LjQ1LTIuMTMuNTEuNTEsMCwwLDEsLjY4LS4wNWMuMjkuMjQuNi40Ny45My43MSwxLjM3LDEsMi45MiwyLjE3LDMuNCwzLjgzYS42OC42OCwwLDAsMSwwLC4xNHY2QS41LjUsMCwwLDEsMjYsMjguNVptLTE5LjUtMWgxOVYyMi4wN2MtLjQxLTEuMy0xLjc5LTIuMzMtMy0zLjIzbC0uNjEtLjQ2QTguMjQsOC4yNCwwLDAsMSwxNiwyMC41YTguMjksOC4yOSwwLDAsMS01Ljg5LTIuMTJsLS42Mi40NmMtMS4yMi45LTIuNiwxLjkzLTMsMy4yM1oiLz48L2c+PC9zdmc+"
                  />
                  {{ task.worker }}
                </div>
                <div v-if="task.img">
                  <svg class="kb-icon margin10" viewBox="0 0 1038 1024">
                    <path
                      d="M939.255172 247.172414c-49.434483-49.434483-120.055172-77.682759-197.737931-77.682759-35.310345 0-70.62069 7.062069-105.931034 21.186207-35.310345 7.062069-63.558621 28.248276-91.806897 56.496552L289.544828 501.406897c-28.248276 28.248276-42.372414 63.558621-42.372414 105.931034s14.124138 77.682759 49.434483 112.993103 70.62069 42.372414 112.993103 42.372414c42.372414 0 84.744828-14.124138 112.993103-42.372414L741.517241 494.344828c7.062069-7.062069 14.124138-14.124138 14.124138-28.248276 0-7.062069-7.062069-21.186207-14.124138-28.248276-7.062069-7.062069-14.124138-14.124138-28.248275-14.124138s-21.186207 14.124138-28.248276 21.186207L459.034483 663.834483c-14.124138 14.124138-35.310345 21.186207-56.496552 21.186207-21.186207 0-42.372414-7.062069-56.496552-21.186207-14.124138-14.124138-21.186207-35.310345-21.186207-56.496552 0-21.186207 7.062069-42.372414 21.186207-56.496552l254.234483-254.234482c35.310345-35.310345 84.744828-56.496552 141.241379-56.496552 49.434483 0 98.868966 21.186207 141.24138 56.496552 35.310345 35.310345 56.496552 84.744828 56.496551 134.17931S918.068966 529.655172 882.758621 564.965517l-275.42069 282.482759c-56.496552 56.496552-127.117241 84.744828-204.8 84.744827-77.682759 0-148.303448-28.248276-204.8-84.744827-56.496552-56.496552-84.744828-127.117241-84.744828-204.8C112.993103 564.965517 141.241379 494.344828 197.737931 437.848276l353.103448-346.041379c7.062069-7.062069 7.062069-21.186207 7.062069-28.248276 0-7.062069-7.062069-21.186207-14.124138-28.248276 0-7.062069-14.124138-14.124138-21.186207-14.124138s-21.186207 7.062069-28.248275 14.124138L141.241379 381.351724c-35.310345 35.310345-63.558621 77.682759-84.744827 120.055173-21.186207 42.372414-28.248276 91.806897-28.248276 141.241379s7.062069 91.806897 28.248276 141.241379c21.186207 42.372414 49.434483 84.744828 77.682758 120.055173 35.310345 35.310345 77.682759 63.558621 120.055173 77.682758 42.372414 21.186207 91.806897 28.248276 141.241379 28.248276s91.806897-7.062069 141.241379-28.248276c49.434483-21.186207 84.744828-42.372414 120.055173-77.682758L932.193103 635.586207c28.248276-28.248276 49.434483-56.496552 63.558621-91.806897 14.124138-35.310345 21.186207-70.62069 21.186207-105.931034 0-70.62069-28.248276-141.241379-77.682759-190.675862z"
                    ></path>
                  </svg>
                  {{ task.img.length }}
                </div>
                <div v-if="task.todo && task.todo.length">
                  <svg class="kb-icon margin10" viewBox="0 0 1026 1024">
                    <path
                      d="M887.75052 998.453222H144.765073c-65.995842 0-119.218295-53.222453-119.218295-119.218295V149.022869c0-65.995842 53.222453-119.218295 119.218295-119.218295h519.451143c21.288981 0 40.449064 17.031185 40.449065 40.449064s-17.031185 40.449064-40.449065 40.449065H144.765073C123.476091 110.702703 106.444906 127.733888 106.444906 149.022869v730.212058c0 21.288981 17.031185 40.449064 40.449065 40.449065h740.856549c21.288981 0 40.449064-17.031185 40.449064-40.449065V447.068607c0-21.288981 17.031185-40.449064 40.449065-40.449064s40.449064 17.031185 40.449064 40.449064v432.16632c-2.128898 65.995842-55.351351 119.218295-121.347193 119.218295z"
                    ></path>
                    <path
                      d="M434.295218 670.602911c-19.160083 0-40.449064-8.515593-53.222453-25.546778L206.503119 413.006237c-12.773389-17.031185-10.644491-42.577963 6.386694-55.351351 17.031185-12.773389 42.577963-10.644491 55.351351 6.386694l168.182952 219.276507L881.363825 202.245322c17.031185-14.902287 42.577963-12.773389 55.351352 4.257797 14.902287 17.031185 12.773389 42.577963-4.257797 55.351351L474.744283 655.700624c-10.644491 8.515593-25.546778 14.902287-40.449065 14.902287z"
                    ></path>
                  </svg>
                  {{ task.todo.length }}
                </div>
                <div v-if="task.tags && task.tags.length">
                  <svg class="kb-icon margin10" viewBox="0 0 1026 1024">
                    <path
                      d="M468.934579 1002.46729c-23.925234 0-50.242991-9.570093-69.383177-28.710281L45.457944 619.663551C26.317757 600.523364 16.747664 576.598131 16.747664 550.280374c0-26.317757 9.570093-50.242991 28.71028-69.383178L488.074766 43.065421c14.35514-14.35514 33.495327-21.53271 55.028038-21.532711l370.841121 19.140187c35.88785 0 64.598131 31.102804 66.990654 66.990654l19.140187 370.841122c0 21.53271-7.17757 40.672897-21.53271 55.028037L535.925234 973.757009c-19.140187 19.140187-43.065421 28.71028-66.990655 28.710281z m71.775701-945.046729l-2.392523 35.88785L98.093458 535.925234c-4.785047 4.785047-7.17757 9.570093-7.17757 16.747663s2.392523 11.962617 7.17757 16.747664l354.093458 354.093458c9.570093 9.570093 23.925234 9.570093 33.495327 0l440.224299-440.224299L909.158879 112.448598l-370.841122-19.140187 2.392523-35.88785z"
                    ></path>
                    <path
                      d="M638.803738 526.35514c-35.88785 0-74.168224-14.35514-100.485981-40.672897s-40.672897-62.205607-40.672897-100.485981 14.35514-74.168224 40.672897-100.485982c55.028037-55.028037 145.943925-55.028037 200.971963 0 26.317757 26.317757 40.672897 62.205607 40.672897 100.485982s-14.35514 74.168224-40.672897 100.485981c-28.71028 26.317757-64.598131 40.672897-100.485982 40.672897z m0-215.327103c-19.140187 0-35.88785 7.17757-50.24299 21.532711-14.35514 14.35514-21.53271 31.102804-21.532711 50.24299s7.17757 35.88785 21.532711 50.242991c26.317757 26.317757 71.775701 26.317757 100.485981 0 14.35514-14.35514 21.53271-31.102804 21.53271-50.242991s-7.17757-35.88785-21.53271-50.24299c-14.35514-14.35514-33.495327-21.53271-50.242991-21.532711z"
                    ></path>
                  </svg>
                  {{ task.tags.length }}
                </div>
                <div v-if="task.links && Object.keys(task.links).length">
                  <img
                    class="kb-icon margin10"
                    src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/Pjxzdmcgdmlld0JveD0iMCAwIDI0IDI0IiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjx0aXRsZS8+PHBhdGggZD0iTTE3LjMsMTMuMzVhMSwxLDAsMCwxLS43LS4yOSwxLDEsMCwwLDEsMC0xLjQxbDIuMTItMi4xMmEyLDIsMCwwLDAsMC0yLjgzTDE3LjMsNS4yOGEyLjA2LDIuMDYsMCwwLDAtMi44MywwTDEyLjM1LDcuNEExLDEsMCwwLDEsMTAuOTQsNmwyLjEyLTIuMTJhNC4xLDQuMSwwLDAsMSw1LjY2LDBsMS40MSwxLjQxYTQsNCwwLDAsMSwwLDUuNjZMMTgsMTMuMDZBMSwxLDAsMCwxLDE3LjMsMTMuMzVaIiBmaWxsPSIjNDY0NjQ2Ii8+PHBhdGggZD0iTTguMTEsMjEuM2E0LDQsMCwwLDEtMi44My0xLjE3TDMuODcsMTguNzJhNCw0LDAsMCwxLDAtNS42Nkw2LDEwLjk0QTEsMSwwLDAsMSw3LjQsMTIuMzVMNS4yOCwxNC40N2EyLDIsMCwwLDAsMCwyLjgzTDYuNywxOC43MmEyLjA2LDIuMDYsMCwwLDAsMi44MywwbDIuMTItMi4xMkExLDEsMCwxLDEsMTMuMDYsMThsLTIuMTIsMi4xMkE0LDQsMCwwLDEsOC4xMSwyMS4zWiIgZmlsbD0iIzQ2NDY0NiIvPjxwYXRoIGQ9Ik04LjgyLDE2LjE4YTEsMSwwLDAsMS0uNzEtLjI5LDEsMSwwLDAsMSwwLTEuNDJsNi4zNy02LjM2YTEsMSwwLDAsMSwxLjQxLDAsMSwxLDAsMCwxLDAsMS40Mkw5LjUyLDE1Ljg5QTEsMSwwLDAsMSw4LjgyLDE2LjE4WiIgZmlsbD0iIzQ2NDY0NiIvPjwvc3ZnPg=="
                  />
                  {{ Object.keys(task.links).length }}
                  <div class="kb-links">
                    <span
                      class="kb-link"
                      v-for="(url, name) of task.links"
                      :key="url"
                    >
                      <a :href="url" target="blank">{{ name }}</a>
                    </span>
                  </div>
                </div>
              </div>
            </li>
          </ul>
          <div
            :style="{ position: 'relative' }"
            class="kb-add-block-container"
            :data-status="kanban.label"
            @click.prevent="addBlock(kanban)"
          >
            <a
              class="kb-add-block"
              :style="{ color: kanban.fontColor || kanban.background }"
              >+ 添加</a
            >
          </div>
        </li>
        <!-- 卡片过多换行 -->
        <!-- <div v-if="index%limit===0" style="flex-wrap:wrap;">123</div> -->
      </div>
    </ul>
  </div>
</template>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  height: 100vh;
  // background: #3e423a;
  display: flex;
  justify-content: center;
  align-items: center;
}
body {
  margin: 0;
  padding: 0;
}
</style>

<script>
import dragula from "dragula";
import { Machine } from "xstate";
import ProgressBar from "vue-simple-progress";

export default {
  name: "KanbanBoard",
  props: {
    tagConfigs: {
      type: Object,
      default() {
        return {};
      },
    },
    limit: {
      type: Number,
      default: 5,
    },
    kanbanList: {
      type: Array,
      required: true,
    },
    blocks: {
      type: Array,
      required: true,
    },
    config: {
      type: Object,
      default: () => ({}),
    },
    stateMachineConfig: {
      type: Object,
      default: null,
    },
    idProp: {
      type: String,
      default: "id",
    },
    statusProp: {
      type: String,
      default: "status",
    },
    cardStyle: {
      type: Object,
      default() {
        return {
          background: "rgba(255,255,255,1)",
        };
      },
    },
  },

  data() {
    return {
      dragInfo: {
        startIndex: -1,
        startKanbanIndex: -1,
        sourceNextEl: null,
        nextEl: null,
        $el: null,
        height: 0,
        putAtLast: false,
      },
      machine: null,
    };
  },

  computed: {
    localBlocks() {
      return this.blocks;
    },
  },
  components: { ProgressBar },
  methods: {
    editTask(task) {
      this.$emit("editTask", task);
    },

    editKanban(kanban) {
      console.log("kanban :>> ", kanban);
      this.$emit("editKanban", kanban);
    },
    //通过元素获取需要移动的元素以及移动的方向
    //2:[1,2,3]，移动2的时候返回3和false，移动3的时候返回2和true
    dealShadow(el, notMove = true) {
      this.clearMargin();
      let thisEl = el;
      let putAtLast = thisEl.nextElementSibling === null;
      const $el = thisEl.nextElementSibling || thisEl.previousElementSibling;
      if (!$el) {
        return;
      }
      this.dragInfo.$el = $el;
      this.dragInfo.putAtLast = putAtLast;
      //腾出位置
      if (notMove) {
        //立即撑出足够的空间
        $el.classList.remove("drag-item");
        setTimeout(() => {
          $el.classList.add("drag-item");
        });
      }
      let height = this.dragInfo.height + "px";
      if (putAtLast) {
        $el.style.marginBottom = height;
        const lastElement = el;
        this.dragInfo.lastElement = lastElement;
      } else {
        $el.style.marginTop = height;
      }
      return putAtLast;
    },

    //清除margin
    clearMargin() {
      if (this.dragInfo.$el) {
        this.dragInfo.$el.style.marginBottom = "10px";
        this.dragInfo.$el.style.marginTop = "10px";
      }
    },

    addBlock(kanban) {
      this.$emit("addBlock", kanban);
    },
    handlerScroll(e) {
      const className = e.target.className;
      if (
        className === "drag-item" ||
        e.target.parentElement.parentElement.className === "drag-item" ||
        e.target.parentElement.className === "drag-item"
      ) {
        return;
      }
      if (e.target.className === "drag-inner-list") {
        if (e.target.scrollHeight > e.target.clientHeight) {
          return;
        }
      }
      /*       if((e.target.parent && e.target.parent.className==='drag-item') ){
          return
        } */
      if (e.deltaY <= 0) {
        this.$emit("previous");
      } else {
        this.$emit("next");
      }
    },
    // getBlocks(status) {
    //   return this.localBlocks.filter(
    //     (block) => block[this.statusProp] === status
    //   );
    // },

    findPossibleTransitions(sourceState) {
      return this.machine.config.states[sourceState].on || {};
    },

    findTransition(target, source) {
      const targetState = target.dataset.status;
      const sourceState = source.dataset.status;
      const possibleTransitions = this.findPossibleTransitions(sourceState);
      return Object.keys(possibleTransitions).find(
        (transition) => possibleTransitions[transition] === targetState
      );
    },

    accepts(block, target, source) {
      if (!this.machine) return true;
      const targetState = target.dataset.status;
      const sourceState = source.dataset.status;
      return Object.values(this.findPossibleTransitions(sourceState)).includes(
        targetState
      );
    },

    allowedTargets(el, source) {
      const block = this.localBlocks.find(
        (b) => b[this.idProp] === el.dataset.blockId
      );
      return this.drake.containers.filter((c) =>
        this.config.accepts(block, c, source)
      );
    },

    forbiddenTargets(el, source) {
      return this.drake.containers.filter(
        (c) => !this.allowedTargets(el, source).includes(c)
      );
    },
  },

  updated() {
    this.drake.containers = this.$refs.list;
    this.drake.mirrorContainer = this.$el;
  },

  mounted() {
    /* 监听滚动 */
    document.addEventListener("mousewheel", this.handlerScroll, true);
    this.config.accepts = this.config.accepts || this.accepts;
    this.config.mirrorContainer = this.$el;
    const _this = this;
    this.drake = dragula(this.$refs.list, this.config)
      .on("drag", (el, source) => {
        //调整下一张卡片的位置，如果没有卡片呢？
        const $el = el;
        let size = $el.getBoundingClientRect();
        _this.dragInfo.height = size.height;
        _this.dragInfo.startIndex = +el.getAttribute("index");
        _this.dragInfo.startKanbanIndex = +source.getAttribute("data-index");
        _this.dragInfo.sourceNextEl = $el.nextElementSibling;
        //  if(_this.kanbanList[cardIndex] && _this.kanbanList[cardIndex].tasks.length>index+1){
        //      //(_this.kanbanList[cardIndex].tasks[index+1]<tBoundingClientRect().height+'px'
        //   }

        this.$emit("drag", el, source);
        el.style.width = size.width + "px";
        el.style.height = size.height + "px";
        el.classList.add("is-moving");
        this.allowedTargets(el, source).forEach((c) =>
          c.classList.add("allowed")
        );
        this.forbiddenTargets(el, source).forEach((c) =>
          c.classList.add("forbidden")
        );
      })
      .on("dragend", (el) => {
        _this.clearMargin();
        if (_this.dragInfo.nextEl) {
          _this.dragInfo.nextEl.classList.remove("drag-item");
          _this.dragInfo.nextEl.style.marginTop = "10px";
          _this.dragInfo.nextEl.style.marginBottom = "10px";
          _this.dragInfo.nextEl.classList.add("drag-item");
        }
        this.$emit("dragend", el);
        el.classList.remove("is-moving");
        this.drake.containers.forEach((c) =>
          c.classList.remove("allowed", "forbidden")
        );
        window.setTimeout(() => {
          el.classList.add("is-moved");
          window.setTimeout(() => {
            el.classList.remove("is-moved");
          }, 600);
        }, 100);
      })
      .on("drop", (block, list, source, sibling) => {
        if (!list) {
          return;
        }
        if (list === source) {
          return;
        }
        //this.dragInfo.startKanbanId =
        this.$emit("drop", block, list, source, sibling);
        let index = 0;
        for (index = 0; index < list.children.length; index += 1) {
          if (list.children[index].classList.contains("is-moving")) break;
        }

        if (this.machine) {
          const transition = this.findTransition(list, source);
          if (!transition) return;
        }
        this.$emit(
          "update-block",
          block.getAttribute("data-block-id"),
          source.getAttribute("data-index"),
          list.getAttribute("data-index"),
          index
        );
      })
      .on("cancel", (el, container, source) => {
        this.$emit("cancel", el, container, source);
      })
      .on("remove", (el, container, source) => {
        this.$emit("remove", el, container, source);
      })
      .on("shadow", (el, container, source) => {
        let kanban = _this.kanbanList[_this.dragInfo.startKanbanIndex];
        if (_this.dragInfo.lastElement) {
          _this.dragInfo.lastElement.style.top = "";
          _this.dragInfo.lastElement = null;
        }
        if (_this.dealShadow(el, false)) {
          //el.style.position = 'relative'
          el.style.top = `${
            el.getBoundingClientRect().top - _this.dragInfo.height
          }px`;
        }
        if (
          _this.dragInfo.sourceNextEl &&
          kanban &&
          kanban.tasks.length > _this.dragInfo.startIndex + 1
        ) {
          _this.dragInfo.sourceNextEl.classList.remove("drag-item");
          _this.dragInfo.sourceNextEl.style.marginTop = _this.dragInfo.height;
          if (_this.dragInfo.sourceNextEl) {
            setTimeout(() => {
              _this.dragInfo.sourceNextEl.classList.add("drag-item");
              _this.dragInfo.sourceNextEl.style.marginTop = "10px";
              _this.dragInfo.sourceNextEl = null;
              // _this.dragInfo.nextEl = $el
            }, 0);
          }
        }
        this.$emit("shadow", el, container, source);
      })
      .on("over", (el, container, source) => {
        this.$emit("over", el, container, source);
      })
      .on("out", (el, container, source) => {
        this.$emit("out", el, container, source);
      })
      .on("cloned", (clone, original, type) => {
        _this.dealShadow(original);
        this.$emit("cloned", clone, original, type);
      });
    this.$emit("init", this.drake);
  },

  created() {
    if (!this.stateMachineConfig) return;
    this.machine = Machine(this.stateMachineConfig);
  },
};
</script>
