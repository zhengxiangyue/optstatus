<template>
    <div>
        <div class="layout">
            <Layout>
                <Header :style="{position: 'fixed', width: '100%', zIndex: '10'}">
                    <Menu mode="horizontal" theme="dark" active-name="1">
                        <div class="layout-logo"></div>
                        <div class="layout-nav">
                            <MenuItem name="1">
                                <Tooltip content="要查询的起始 10 位Receipt Number">
                                    <Input :disabled="processing" placeholder="Start ID" v-model="start_id"></Input>
                                </Tooltip>
                            </MenuItem>
                            <MenuItem name="2">
                                <Tooltip content="要查询的结束 10 位Receipt Number">
                                    <Input :disabled="processing" placeholder="End ID" v-model="end_id"></Input>
                                </Tooltip>
                            </MenuItem>
                            <MenuItem name="3">
                                <Tooltip content="查询步长">
                                    <Input :disabled="processing" style="width: 50px" v-model="step" placeholder="Step"></Input>
                                </Tooltip>
                            </MenuItem>
                            <MenuItem name="4">
                                <Button type="primary" :disabled="processing" @click="start">Get Data</Button>
                                <Button type="warning" :disabled="!processing" @click="user_stop = true">Stop</Button>
                            </MenuItem>
                            <!--<MenuItem name="2">-->
                                <!--<Icon type="ios-keypad"></Icon>-->
                                <!--Item 2-->
                            <!--</MenuItem>-->
                            <!--<MenuItem name="3">-->
                                <!--<Icon type="ios-analytics"></Icon>-->
                                <!--Item 3-->
                            <!--</MenuItem>-->
                            <!--<MenuItem name="4">-->
                                <!--<Icon type="ios-paper"></Icon>-->
                                <!--Item 4-->
                            <!--</MenuItem>-->
                        </div>
                    </Menu>
                </Header>
                <Content :style="{margin: '88px 20px 0', background: '#fff', minHeight: '500px'}">
                    <Card>
                        <Col>
                            <Page :total="dataList.length" :page-size="pageSize" show-total  @on-change="changePage"></Page>
                        </Col>
                    </Card>
                    <Card>
                        <Col>
                            <Table :columns="columns" :data="showingData"></Table>
                        </Col>
                    </Card>
                </Content>
            </Layout>
        </div>
    </div>
</template>

<script>

    const request = require('request');

    export default {
        name: 'landing-page',
        data: function () {
            return {
                pageSize: 100,
                start_id: 1990270000,
                end_id: 1990300000,
                step: 1,

                processing: false,

                user_stop: false,

                dataList: [],
                columns: [
                    {
                        title: 'case id',
                        key: 'id',
                        width: 180,
                        align: 'center',
                        sortable: true,
                    },
                    {
                        title: 'status',
                        key: 'status',
                        width: 350,
                        align: 'center',
                        sortable: true,
                    },
                    {
                        title: 'description',
                        key: 'description',
                    }],
                currentIndex: 1,
            };
        },

        mounted: function () {
            // this.getSingleData();
        },

        computed: {
            showingData () {
                let _start = ( this.currentIndex - 1 ) * this.pageSize;
                let _end = this.currentIndex * this.pageSize;
                return this.dataList.slice(_start,_end);
            },
        },

        methods: {

            changePage(index){
                this.currentIndex = index;
            },

            remove_html_tags(text) {
                return text.replace(/<[^>]*>?/gm, '');
            },

            start() {

                if (isNaN(this.start_id) || isNaN(this.end_id)) {
                    this.$Message.warning({
                        content: "Id should be a number of length 10",
                        duration: 5,
                    });
                    return;
                }

                if (Number(this.start_id).toString().length != 10 || (Number(this.end_id).toString().length != 10)) {
                    this.$Message.warning({
                        content: "Id should be of length 10",
                        duration: 5,
                    });
                    return;
                }

                this.processing = true;
                this.user_stop = false;
                this.dataList = [];
                this.case_id = Number(this.start_id);
                this.getSingleData();
            },

            getSingleData() {
                let that = this;

                if (that.user_stop || Number(that.case_id) > Number(that.end_id)) {
                    that.processing = false;
                    return;
                }

                request('https://egov.uscis.gov/casestatus/mycasestatus.do?appReceiptNum=YSC' + that.case_id, function (error, response, body) {
                    // console.log('statusCode:', response && response.statusCode); // Print the response status code if a response was received
                    let start_index;
                    let end_index;

                    try {
                        start_index = body.indexOf('<div class="rows text-center">');
                        end_index = body.indexOf('<div class="col-lg-12 case-status-from form3 mt40 clr">');
                    } catch (e) {
                        return;
                    }


                    let key_data = body.substring(start_index, end_index);

                    let status_start = key_data.indexOf('<h1>');
                    let status_end = key_data.indexOf('</h1>');
                    let status = that.remove_html_tags(key_data.substring(status_start, status_end));

                    let description_start = key_data.indexOf('<p>');
                    let description_end = key_data.indexOf('</p>');
                    let description = that.remove_html_tags(key_data.substring(description_start, description_end));

                    console.log(status);

                    that.dataList.push({
                        id: 'YSC' + that.case_id,
                        status: status,
                        description: description,
                    });

                    that.case_id = Number(that.case_id) + Number(that.step);
                    that.getSingleData();
                });
            },

        }
    }
</script>

<style>
    @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

    * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    body {
        font-family: 'Source Sans Pro', sans-serif;
    }

    #wrapper {
        background: radial-gradient(
                ellipse at top left,
                rgba(255, 255, 255, 1) 40%,
                rgba(229, 229, 229, .9) 100%
        );
        height: 100vh;
        padding: 60px 80px;
        width: 100vw;
    }

    #logo {
        height: auto;
        margin-bottom: 20px;
        width: 420px;
    }

    main {
        display: flex;
        justify-content: space-between;
    }

    main > div {
        flex-basis: 50%;
    }

    .left-side {
        display: flex;
        flex-direction: column;
    }

    .welcome {
        color: #555;
        font-size: 23px;
        margin-bottom: 10px;
    }

    .title {
        color: #2c3e50;
        font-size: 20px;
        font-weight: bold;
        margin-bottom: 6px;
    }

    .title.alt {
        font-size: 18px;
        margin-bottom: 10px;
    }

    .doc p {
        color: black;
        margin-bottom: 10px;
    }

    .doc button {
        font-size: .8em;
        cursor: pointer;
        outline: none;
        padding: 0.75em 2em;
        border-radius: 2em;
        display: inline-block;
        color: #fff;
        background-color: #4fc08d;
        transition: all 0.15s ease;
        box-sizing: border-box;
        border: 1px solid #4fc08d;
    }

    .doc button.alt {
        color: #42b983;
        background-color: transparent;
    }
</style>
