<template>
  <div class="row">
    <div class="col-md-12">
      <h1>Jail [{{jail_name}}]</h1>
      <div v-if="jail_data['error']">
        <div class="alert alert-danger alert-dismissible fade show" role="alert">
          Cannot connect to API - is it running?
          <button type="button" class="close" data-dismiss="alert" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
      </div>
      <div class="card mb-3"  v-for="data in jail_data">
        <div class="card-body">
          <p class="card-text">{{data.log_file}}</p>
        </div>
      </div>
    </div>
    <div class="col-sm-3" v-for="data in jail_data">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">Current bans</h5>
          <h1 class="text-center">{{data.bans_current}}</h1>
        </div>
      </div>
    </div>
    <div class="col-sm-3" v-for="data in jail_data">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">Total bans</h5>
          <h1 class="text-center">{{data.bans_total}}</h1>
        </div>
      </div>
    </div>
    <div class="col-sm-3" v-for="data in jail_data">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">Current fails</h5>
          <h1 class="text-center">{{data.fails_current}}</h1>
        </div>
      </div>
    </div>
    <div class="col-sm-3" v-for="data in jail_data">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">Total fails</h5>
          <h1 class="text-center">{{data.fails_total}}</h1>
        </div>
      </div>
    </div>
    <div class="col-sm-6 mt-3">
      <h4 class="float-left">Banned IPs</h4>
      <div class="float-right p-1"><a href="#" @click="refreshData">Refresh <font-awesome-icon icon="sync-alt"/></a></div>
      <input type="text" v-model="search" class="form-control" placeholder="Search IP">
      <div class="mt-2">
        <div v-for="(ip, index) in filterIPs" class="delObjects p-1 m-1">
            {{ip}} <a href="#" v-on:click.prevent="deleteObject(ip, index)">&times;</a>
        </div>
      </div>
    </div>
    <div class="col-sm-6 mt-3">
      <h4>Ban IP</h4>
      <notifications group="banStatus" />
      <form class="form-inline" v-on:submit="banIP" action="#">
        <input type="text" v-model="banningIP"  class="form-control mb-2 mr-sm-2" placeholder="xxx.xxx.xxx.xxx">
        <button type="submit" class="btn btn-secondary mb-2">Ban</button>
      </form>
    </div>
    <div class="col-sm-6 mt-3">
      <h4 class="float-left">Ignored IPs</h4>
      <div class="float-right p-1"><a href="#" @click="refreshData">Refresh <font-awesome-icon icon="sync-alt"/></a></div>
      <input type="text" v-model="search_ignore" class="form-control" placeholder="Search ignore IP">
      <div class="mt-2">
        <div v-for="(ip, index) in ignoreIPs" class="delObjects p-1 m-1">
            {{ip}} <a href="#" v-on:click.prevent="deleteIgnoreObject(ip, index)">&times;</a>
        </div>
      </div>
    </div>
    <div class="col-sm-6 mt-3">
      <h4>Ignore IP</h4>
      <notifications group="ignoreStatus" />
      <form class="form-inline" v-on:submit="ignoreIP" action="#">
        <input type="text" v-model="ignoringIP"  class="form-control mb-2 mr-sm-2" placeholder="xxx.xxx.xxx.xxx">
        <button type="submit" class="btn btn-secondary mb-2">Ignore IP</button>
      </form>
    </div>
  </div>
</template>
<script>
import axios from 'axios'

require('../../static/main.css')

export default {
  name: 'Jails',
  components: {
    axios
  },
  data () {
    return {
      jail_data: "",
      jail_name: this.$route.params.jailname,
      banlist: "",
      ignorelist: "",
      search: "",
      search_ignore: "",
      banningIP: "",
      banStatus: "",
      ignoreingIP: "",
      ignoreStatus: ""
    }
  },
  created () {
    this.getJails();
    this.getIgnoreIp();
  },
  watch: {
    '$route': 'getJails'
  },
  methods: {
    getJails () {
      const url = `${window.location.protocol}//${window.location.hostname}/api/jail/${this.jail_name}`;
      axios.get(url)
        .then((resp) => {
          this.jail_data = resp.data;
          for (let key in resp.data) {
              this.banlist = resp.data[key].bans_iplist;
            }
        })
        .catch((err) => {
          console.log(err);
        })
    },
    getIgnoreIp () {
      const url = `${window.location.protocol}//${window.location.hostname}/api/jail/${this.jail_name}/ignoreip`;
      axios.get(url)
        .then((resp) => {
          this.jail_data = resp.data;
          for (let key in resp.data) {
              this.ignorelist = resp.data.details;
            }
        })
        .catch((err) => {
          console.log(err);
        })
    },
    refreshData() {
      this.getJails();
      this.getIgnoreIp();
    },
    ValidateIPaddress(ipaddress) {
        const octetPattern = "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)";
        const regex = new RegExp(`^${octetPattern}\\.${octetPattern}\\.${octetPattern}\\.${octetPattern}$`);
        return regex.test(ipaddress);
    },
    banIP: function(event){
      let validIP = this.ValidateIPaddress(this.banningIP)
      if (this.banningIP) {
        if (validIP) {
          // POST ban
          const url = `${window.location.protocol}//${window.location.hostname}/api/ban`;
          axios.post(url, {
              'ip': this.banningIP,
              'jail': this.jail_name,
            })
            .then(function (response) {
              console.log(response);
            })
          .catch(function (error) {
            console.log(error);
          });
          // refresh jail needs to wait a lil?
          this.getJails();
          // notify
          this.$notify({
            group: 'banStatus',
            title: 'Success!',
            text: '<b>' + this.banningIP + '</b>' + ' has been banned in jail <b>' + this.jail_name + '</b>',
            type: 'success'
          });
        } else {
          this.$notify({
            group: 'banStatus',
            title: 'Error!',
            text: '<b>' + this.banningIP + '</b>' + ' is not a valid IP-address.',
            type: 'error'
          });
        }
      } else {
        this.$notify({
          group: 'banStatus',
          title: 'Error!',
          text: 'You have to input something dummy.',
          type: 'warn'
        });        
      }
    },   
    ignoreIP: function(event){
      let validIP = this.ValidateIPaddress(this.ignoringIP)
      if (this.ignoringIP) {
        if (validIP) {
          // POST ignoring IP
          const url = `${window.location.protocol}//${window.location.hostname}/api/jail/${this.jail_name}/addignoreip`;
          axios.post(url, {
              'ip': this.ignoringIP,
              'jail': this.jail_name,
            })
            .then(function (response) {
              console.log(response);
            })
          .catch(function (error) {
            console.log(error);
          });
          // refresh jail needs to wait a lil?
          this.getIgnoreIp();
          // notify
          this.$notify({
            group: 'ignoreStatus',
            title: 'Success!',
            text: '<b>' + this.ignoringIP + '</b>' + ' has been ignored in jail <b>' + this.jail_name + '</b>',
            type: 'success'
          });
        } else {
          this.$notify({
            group: 'ignoreStatus',
            title: 'Error!',
            text: '<b>' + this.ignoringIP + '</b>' + ' is not a valid IP-address.',
            type: 'error'
          });
        }
      } else {
        this.$notify({
          group: 'ignoreStatus',
          title: 'Error!',
          text: 'You have to input something dummy.',
          type: 'warn'
        });
      }
    },
    deleteObject: function(ip, index) {
      this.$delete(this.banlist, index);
      // POST unban
      const url = `${window.location.protocol}//${window.location.hostname}/api/unban`;
      axios.post(url, {
          'ip': ip,
          'jail': this.jail_name,
        })
        .then(function (response) {
          console.log(response);
        })
      .catch(function (error) {
        console.log(error);
      });
      // refresh jail
      this.getJails()
      // notify
      this.$notify({
        group: 'banStatus',
        title: 'Success!',
        text: '<b>' + ip + '</b>' + ' has been unbanned in jail <b>' + this.jail_name + '</b>',
        type: 'success'
      });
    },
    deleteIgnoreObject: function(ip, index) {
      this.$delete(this.ignorelist, index);
      // POST delete ignore IP
      const url = `${window.location.protocol}//${window.location.hostname}/api/jail/${this.jail_name}/delignoreip`;
      axios.post(url, {
          'ip': ip,
          'jail': this.jail_name,
        })
        .then(function (response) {
          console.log(response);
        })
      .catch(function (error) {
        console.log(error);
      });
      // refresh jail
      this.getJails()
      // notify
      this.$notify({
        group: 'ignoreStatus',
        title: 'Success!',
        text: '<b>' + ip + '</b>' + ' has been ignored in jail <b>' + this.jail_name + '</b>',
        type: 'success'
      });
    },
  },
  computed: {
    filterIPs: function() {
      let filtered = this.banlist;
      if (this.search) {
        filtered = this.banlist.filter(
          m => m.toLowerCase().indexOf(this.search.toLowerCase()) > -1,
        );
      }
      return filtered;
    },
    ignoreIPs: function() {
      let ignored = this.ignorelist;
      if (this.search_ignore) {
        ignored = this.ignorelist.filter(
          m => m.toLowerCase().indexOf(this.search.toLowerCase()) > -1,
        );
      }
      return ignored;
    },
  }
}
</script>
