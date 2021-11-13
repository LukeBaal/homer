<template>
  <div>
    <div class="card" :class="item.class">
      <a :href="item.url" :target="item.target" rel="noreferrer">
        <div class="card-content">
          <div class="media">
            <div v-if="item.logo" class="media-left">
              <figure class="image is-48x48">
                <img :src="item.logo" :alt="`${item.name} logo`" />
              </figure>
            </div>
            <div v-if="item.icon" class="media-left">
              <figure class="image is-48x48">
                <i style="font-size: 35px" :class="['fa-fw', item.icon]"></i>
              </figure>
            </div>
            <div class="media-content">
              <p class="title is-4">{{ item.name }}</p>
              <p class="subtitle is-6">
                <template v-if="item.subtitle">
                  {{ item.subtitle }}
                </template>
                <template v-else-if="api && api.status.length > 0 && api.status == 'enabled'">
                  <span>{{ api.running }} Running</span>
                  {{ " - " }}
                  <span>{{ api.stopped }} Stopped</span>
                </template>
              </p>
            </div>
            <div v-if="api && api.status == 'disabled'" class="status" :class="api.status">
              {{ api.status }}
            </div>
          </div>
          <div class="tag" :class="item.tagstyle" v-if="item.tag">
            <strong class="tag-text">#{{ item.tag }}</strong>
          </div>
        </div>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  name: "Portainer",
  props: {
    item: Object,
  },
  data: () => ({
    api: {
      status: "",
      running: 0,
      stopped: 0,
      endpoints: []
    },
  }),
  async created() {
    const token = await this.fetchToken();
    this.fetchStatus(token);
  },
  methods: {
    fetchToken: async function () {
      try {
        const url = `${this.item.url}/api/auth`;
        const res = await fetch(url, {
          method: "POST",
          credentials: "include",
          body: JSON.stringify({
            username: this.item.username,
            password: this.item.password,
          })
        });
        const data = await res.json();
        return data.jwt;
      } catch (e) {
        console.log(e);
      }
    },
    fetchStatus: async function (token) {
      try {
        const url = `${this.item.url}/api/endpoints?limit=100&start=0`;
        const res = await fetch(url, {
          credentials: "include",
          headers: {
            "Authorization": `Bearer ${token}`,
            "Accept": "application/json"
          }
        });
        const endpoints = await res.json();

        if (endpoints.length === 0 || endpoints[0].Snapshots.length === 0) {
          this.api = {
            status: "disabled"
          }
          return;
        }
        const endpoint = endpoints[0];
        this.api = {
          status: endpoint.Status === 1 ? "enabled" : "disabled",
          running: endpoint.Snapshots[0].RunningContainerCount,
          stopped: endpoint.Snapshots[0].StoppedContainerCount
        }
      } catch (e) {
        console.log(e);
        this.api = {
          status: "disabled"
        }
      }
    },
  },
};
</script>

<style scoped lang="scss">
.media-left img {
  max-height: 100%;
}
.status {
  font-size: 0.8rem;
  color: var(--text-title);

  &.enabled:before {
    background-color: #94e185;
    border-color: #78d965;
    box-shadow: 0 0 5px 1px #94e185;
  }

  &.disabled:before {
    background-color: #c9404d;
    border-color: #c42c3b;
    box-shadow: 0 0 5px 1px #c9404d;
  }

  &:before {
    content: " ";
    display: inline-block;
    width: 7px;
    height: 7px;
    margin-right: 10px;
    border: 1px solid #000;
    border-radius: 7px;
  }
}
</style>
