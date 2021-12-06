<template>
    <div class="row justify-content-center">
        <div class="col-md-12">
           <div class="card-body">
                <input v-if="userNotJoined" class="form-control" type="text" v-model="username" v-on:keyup.13="connectClientWithUsername" placeholder="Your username">
                <div class="col-md-4 channel-list" v-show="connected">
                    <ul>
                        <li v-for="(channel) in tc.channelArray" :key="channel.id" ref="channelList" :data-sid="channel.sid">
                            <a href="#!" @click="selectChannel(channel)"> {{channel.friendlyName}} </a>
                        </li>

                        <a href="#!" class="btn btn-info" @click="createChannel"> Add Channel</a>
                        <input v-if="showAddChannelInput" class="form-control" type="text" v-model="newChannel" v-on:keyup.13="handleNewChannelInputKeypress" placeholder="New Channel">
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template> 

<script>
var moment = require('moment');

export default {
    data: function () {
        return {
        tc: {
            accessManager: null,
            messagingClient: null,
            channel: [],
            generalChannel: null,
            username: '',
            channelArray: [],
            currentChannel: null,
            activeChannelIndex: null,
            messagesArray: [],
        },
        username: '',
        connected: false,
        selected: false,
        showMessages: false,
        moment: moment,
        message: null,
        userNotJoined: true,
        newChannel: '',
        showAddChannelInput: false,
        notification: false,
        notificationMsg: '',
        }
    },

    methods:{

        connectClientWithUsername(){
            if (this.username == '') {
                alert('Username cannot be empty');
                return;
            }
            this.tc.username = this.username;
            
            this.fetchAccessToken(this.tc.username, this.connectMessagingClient);
        },
        fetchAccessToken(username, handler) {
            let vm = this;
            axios.post('https://dev.chat-app.com/token', {
                identity: this.tc.username,
                device: 'browser',
                ttl: 3600
            })
            .then(function (response) {
                handler(response.data);
                // console.log(response.data);
                vm.username = '';
            })
            .catch(function (error) {
                console.log(error);
            });
        },
        connectMessagingClient(token) {
            // Initialize the Chat messaging client
            let vm = this;

            this.tc.accessManager = new Twilio.AccessManager(token);
            console.log(token);
            // new Twilio.Conversation.Client.create(token)
            new Twilio.Chat.Client.create(token).then(function(client) {
                    console.log(client);
                    
                //     // vm.tc.messagingClient = client;
                //     // vm.updateConnectedUI();
                //     // vm.loadChannelList(vm.joinGeneralChannel);
                //     // vm.tc.messagingClient.on('channelAdded', _.throttle(vm.loadChannelList));
                //     // vm.tc.messagingClient.on('channelRemoved', _.throttle(vm.loadChannelList));
                //     // vm.tc.messagingClient.on('tokenExpired', vm.refreshToken);
            });
        },
        updateConnectedUI(){
            this.connected = true;
        },
        refreshToken(){
            this.fetchAccessToken(this.tc.username, vm.setNewToken);
        },
        setNewToken(tokenResponse) {
            this.tc.accessManager.updateToken(tokenResponse.token);
        },
       handleNewChannelInputKeypress(event) {
            let vm = this;
            if (this.newChannel == '') {
                alert('Channel name cannot be empty');
                return;
            }
            this.tc.messagingClient.createChannel({
                friendlyName: this.newChannel
            }).then(function(channel) {
                console.log('Created channel');
                vm.loadChannelList(channel);
            }).then(this.hideAddChannelInput);
            this.newChannel = '';
        },

        hideAddChannelInput(){
            this.showAddChannelInput = false;
        },

        createChannel(){
            this.showAddChannelInput = true;
        },
   }
}
</script>