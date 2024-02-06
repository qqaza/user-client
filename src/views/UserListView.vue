<!-- UserListView.vue-->
<template>
    <div class="container">
        <h1>전체 회원 조회</h1>
        <table class="table">
            <caption>Total : {{ count }}</caption>
            <thead>
                <tr>
                    <th>No.</th>
                    <th>ID</th>
                    <th>이름</th>
                    <th>성별</th>
                    <th>가입날짜</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(list, idx) in userList" v-bind:key="idx"
                    v-on:click="goToUserInfo(list.user_id)" >  
                    <td >{{ list.user_no }}</td>
                    <td>{{ list.user_id }}</td>
                    <td v-text="list.user_name" />
                    <td>{{ list.user_gender }}</td>
                    <td v-text="list.join_data" />
                    <!-- v-text 사용가능.-->
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    data(){
        return {
            userList : []
        }
    },
    computed : {
        //기존의 있는 값으로 새로운 값을 생성. (없는 값을 만들어내는것)
        count() {
            return this.userList.length;
        }
    },
    //이미 존재하는 것을 감시하는 역할 ==> return이 없음. 단순 프로세스
    watch : { 
        userList(newQuestion, oldQuestion){
            console.log('이전 : ',oldQuestion);
            alert('데이터가 변경되었습니다,');
            console.log('이후 : ', newQuestion);
        }
    },
    created(){ //데이터가 생성되기 전에 처리되어야 할 부분.
        
    this.getUserList(); //비동기 작업.
    },
    methods : {
        async getUserList(){
            // 동기작업.
            let result = await axios.get('/api/users')
                              .catch(err => console.log(err));
            let list = result.data;
            this.userList = list;
        },
        goToUserInfo(userId){
            //등록된 라우터 요청하는 메소드. /body가 없음(get 방식)
            this.$router.push({ path : '/userInfo', query : { "userId" : userId }}); 
           // this.$router.push({ name : 'userInfo', query : { "userId" : userId }}); 
        }
    }
}
</script>
