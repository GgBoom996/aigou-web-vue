<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.name" placeholder="姓名"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleViewProperties">显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOnSale">上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOffSale">下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="标题" width="150" sortable>
			</el-table-column>
			<el-table-column prop="subName" label="副标题" width="150" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="商品品牌" width="120" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="110" sortable :formatter="formatOnSaleTime">
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间" width="110" sortable :formatter="formatOffSaleTime">
			</el-table-column>
			<el-table-column prop="state" label="状态" width="120" sortable>
				<template scope="scope">
					<span v-if="scope.row.state==1" style="color:green">上架</span>
					<span v-else style="color:gray">下架</span>
				</template>
			</el-table-column>
			<el-table-column label="操作" min-width="120">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别">
					<el-radio-group v-model="editForm.sex">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="editForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="editForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="editForm.addr"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="addForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型" prop="productTypeId">
					<el-input v-model="addForm.productTypeId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="品牌" prop="brandId">
					<el-input v-model="addForm.brandId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="媒体属性" prop="brandId">
					<el-upload
							class="upload-demo"
							action="http://localhost:1299/services/common/fastdfs"
							:on-remove="handleRemoveImg"
							:on-success="handleSuccessImg"
							:file-list="mediasList"
							list-type="picture">
						<el-button size="small" type="primary">点击上传</el-button>
						<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
					</el-upload>
				</el-form-item>
				<el-form-item label="描述" prop="description">
					<el-input v-model="addForm.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品详情" prop="richContent">
					<!--<el-input type="textarea" v-model="addForm.richContent" auto-complete="off"></el-input>-->
					<quill-editor
							v-model="addForm.richContent"
							ref="myQuillEditor"
					>
					</quill-editor>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--显示属性维护的dialog-->
		<el-dialog title="显示属性" v-model="viewPropertiesDialogVisible" size="tiny">

			<el-card class="box-card">
				<el-row v-for="vp in viewProperties" style="margin-top: 5px">
					<el-col :span="6">
						{{vp.specName}}:
					</el-col>
					<el-col :span="18">
						<el-input type="text" v-model="vp.value"/>
					</el-col>
				</el-row>
			</el-card>
			<span slot="footer" class="dialog-footer">
				<el-button @click="dialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="submitViewProperties">确 定</el-button>
			</span>
		</el-dialog>

		<!--sku属性维护的dialog-->
		<el-dialog title="SKU属性" v-model="skuPropertiesDialogVisible" size="small">

			<el-card class="box-card">
				<div v-for="(sku,skuIndex) in skuProperties">
					{{sku.specName}}
					<!--index 是循环的次数  从1开始-->
					<el-row v-for="index in sku.options.length+1" style="margin-top: 5px">
						<el-col :span="19">
							<el-input type="text" v-model="sku.options[index-1]"></el-input>
						</el-col>
						<el-col :span="4" style="margin-left: 5px;">
							<el-button icon="delete" size="small" @click="deleteSkuProperty(skuIndex,index-1)"></el-button>
						</el-col>
					</el-row>
				</div>
			</el-card>

			<el-table :data="skus">
				<el-table-column type="index" width="60">
				</el-table-column>
				<el-table-column v-for="(header,key) in this.skus[0]" v-if="key!='indexes'"  :prop="key" :label="key" min-width="150" sortable>
					<template scope="scope" v-if="key=='price'">
						<el-input v-model="scope.row.price"></el-input>
					</template>
					<template scope="scope" v-else-if="key=='store'">
						<el-input v-model="scope.row.store"></el-input>
					</template>
				</el-table-column>

			</el-table>

			<span slot="footer" class="dialog-footer">
				<el-button @click="dialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="submitSkuProperties">确 定</el-button>
			</span>
		</el-dialog>

	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
			    //年龄[萝莉 御姐] 肤色[土豪金 绿巨人]

			    skus:[{
			        "年龄":"萝莉",
					"肤色":"土豪金",
					price:0,
					store:0
				},{
                    "年龄":"御姐",
                    "肤色":"土豪金",
                    price:0,
                    store:0
                },{
                    "年龄":"萝莉",
                    "肤色":"绿巨人",
                    price:0,
                    store:0
                },{
                    "年龄":"御姐",
                    "肤色":"绿巨人",
                    price:0,
                    store:0
                }],
                skuProperties:[],
                skuPropertiesDialogVisible:false,
			    viewProperties:[],
                viewPropertiesDialogVisible:false,
			    mediasList:[],//媒体属性图片路径数组
				mediasIds:[],//媒体属性图片的fileId的数组
				filters: {
					name: ''
				},
				pageSize:10,
				products: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中行

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					id: 0,
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入标题', trigger: 'blur' }
					],
                    subName: [
                        { required: true, message: '请输入副标题', trigger: 'blur' }
                    ]
				},
				//新增界面数据
				addForm: {
					name: '',
					subName:'',
					productTypeId:null,
					brandId:null,
					description:'',
					richContent:''
				}

			}
		},
		methods: {
            submitSkuProperties(){
                let para = {};
                para.productId = this.sels[0].id;
                para.skus = this.skus;
                para.skuProperties = this.skuProperties;
                this.$http.post("/product/product/updateSkuProperties",para)
					.then(res=>{
                        let ajaxResult = res.data;
                        if(ajaxResult.success){
                            this.$message({
                                message:"操作成功!",
                                type:"success"
                            })
                            this.skuPropertiesDialogVisible = false;
                        }else{
                            this.$message({
                                message:ajaxResult.message,
                                type:"error"
                            })
                        }
					})
			},
            deleteSkuProperty(skuIndex,index){
                //js中数组删除指定位置的元素 splice(index,len,[item])  index操作的位置  len删除的个数  item..插入的元素
				this.skuProperties[skuIndex].options.splice(index,1);
			},
		    //提交修改的显示属性
            submitViewProperties(){
                let para = {};
                para.productId = this.sels[0].id;
                para.viewProperties = JSON.stringify(this.viewProperties);
				this.$http.post("/product/product/updateViewProperties",para)
					.then(res=>{
					    let ajaxResult = res.data;
					    if(ajaxResult.success){
					        this.$message({
								message:"操作成功!",
								type:"success"
							})
							this.viewPropertiesDialogVisible = false;
						}else{
                            this.$message({
                                message:ajaxResult.message,
                                type:"error"
                            })
						}
					})
			},
		    //显示属性维护
            handleViewProperties(){
                //必须选中一行数据才能修改显示属性
				if(this.sels.length!=1){
				    this.$message({
						message:"亲，只能选中一行进行操作！",
						type:"warning"
					})
					return;
				}
				//列出商品的显示属性
				//（1）之前已经设置过显示属性的商品，则要查询t_product_ext并做回显
				//(2)之前没有设置过显示属性的商品，t_product_ext的viewProperties为null，则根据商品类型查询
				//属性表，获取到要维护的属性，生成一个动态表单，用户填值

				//发请求到后台获取商品的显示属性的信息
				let productId = this.sels[0].id;
				this.$http.get("/product/product/getViewProperties?productId="+productId)
					.then(res=>{
					    this.viewProperties = res.data;
					})

				this.viewPropertiesDialogVisible = true;

			},
			//sku属性维护
            handleSkuProperties(){
                //必须选中一行数据才能修改显示属性
                if(this.sels.length!=1) {
                    this.$message({
                        message: "亲，只能选中一行进行操作！",
                        type: "warning"
                    })
                    return;
                }
                //查询sku属性和选项
                let productId = this.sels[0].id;
                this.$http.get("/product/product/getSkuProperties?productId="+productId)
                    .then(res=>{
                        this.skuProperties = res.data;
                    })

                this.skuPropertiesDialogVisible = true;
			},
			//上架
            handleOnSale(){},
			//下架
            handleOffSale(){},
            handleSuccessImg(response, file, fileList){
                let fileId = response.restObj;//从响应中获取到fileId
                this.mediasList = fileList;
            },
            //删除图片
            handleRemoveImg(file, fileList){
                console.debug("file",file)
                console.debug("fileList",fileList)
				//meidasIds数组中删除对应的元素

                this.mediasList = fileList;
            },
			//日期显示转换
            formatOnSaleTime: function (row, column) {
                return this.formatDate(row.onSaleTime)
			},
            formatOffSaleTime: function (row, column) {
                return this.formatDate(row.offSaleTime)
            },
			formatDate(time){
                if(!time){
                    return null;
				}
                let date = new Date(time);
                let year = date.getFullYear();
                let month = date.getMonth()+1;
                let day = date.getDate();
                let hour = date.getHours();
                let minute = date.getMinutes();
                let second = date.getSeconds();
                let timeStr = year+"-"+this.numberFormat(month)+"-"+this.numberFormat(day)
					+" "+this.numberFormat(hour)+":"+this.numberFormat(minute)+":"+this.numberFormat(second);
                return timeStr;
			},
			numberFormat(num){
                if(num<10){
                    return "0"+num;
				}
				return num;
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取商品列表
			getProducts() {
				let para = {
					pageNum: this.page,
					pageSize:this.pageSize
				};
				this.listLoading = true;
				//NProgress.start();
				this.$http.post("/product/product/json",para)
					.then(res=>{
                        this.listLoading = false;
					    let pageList = res.data;
					    this.total = pageList.total;
					    this.products = pageList.rows;
					})


				/*getUserListPage(para).then((res) => {
					this.total = res.data.total;
					this.products = res.data.products;
					this.listLoading = false;
					//NProgress.done();
				});*/
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { id: row.id };
					removeUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getProducts();
					});
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
                    name: '',
					subName:'',
					productTypeId:null,
					brandId:null,
					description:'',
					richContent:''
                }
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editUser(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getProducts();
							});
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
                            //[1,2,3].join(",")   ==>     "1,2,3"
                            para.medias = this.mediasIds.join(",");
							let productExt = {}
							productExt.description = this.addForm.description
							productExt.richContent = this.addForm.richContent
							para.productExt = productExt
							this.$http.post("/product/product/add",para)
								.then(res=>{
                                    this.addLoading = false;
                                    if(res.data.success){
                                        this.$message({
                                            message: '提交成功',
                                            type: 'success'
                                        });
                                        this.$refs['addForm'].resetFields();
                                        this.addFormVisible = false;
                                        this.getProducts();
									}else{
                                        this.$message({
                                            message: res.data.message,
                                            type: 'error'
                                        });
									}
								})
							/*addUser(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getProducts();
							});*/
						});
					}
				});
			},
			selsChange: function (sels) {//sels 选中行的数据
                console.debug("sels",sels)
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getProducts();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getProducts();
		},
		watch:{
		    'mediasList':function(newData){
		        //监听mediasList的变化，重新为mediasIds赋值
				let newMediasIds = [];
				for(var i=0;i<newData.length;i++){
                    newMediasIds.push(newData[i].response.restObj);
				}
				this.mediasIds = newMediasIds;

				console.debug("mediasIds",this.mediasIds)
			},
			'skuProperties':{
		        handler(newData){
                    //根据skuProperties更改skus
                    //skuProperties [{id:1,specName:"年龄",options:["萝莉","御姐"]},{id:2,specName:"肤色",options:["金","绿"]}]
                    //skus [{"年龄":"萝莉","肤色":"金"},{"年龄":"萝莉","肤色":"绿"},{"年龄":"御姐","肤色":"金"}，{"年龄":"御姐","肤色":"绿"}]
					//过滤掉options为空的sku属性
                    let newSkuProperties = this.skuProperties.filter(item=>item.options.length>0)
					let result = newSkuProperties.reduce((lastResult,currentEle)=>{ //lastResult [{}]
                        let newArr = []
                        lastResult.forEach(e1=>{ //e1  {}

							for(var i=0;i<currentEle.options.length;i++){ //currentEle.options  ["萝莉","御姐"]
							    let e2 = currentEle.options[i]; //e2 = “萝莉”
                                let obj = {}
                                Object.assign(obj,e1)
                                obj[currentEle.specName] =e2; // obj {"年龄"："萝莉"}
								obj.price = 0;
								obj.store = 0;
								obj.indexes = (obj.indexes?obj.indexes:'')+i+"_";
                                newArr.push(obj)   //[{"年龄"："萝莉",indexes:"0_"}]
							}
                        })
                        return newArr;
                    },[{}])
                    //console.debug("result",result);
					this.skus = result;
				},
				deep:true //深度监听
			}

		}
	}

</script>

<style scoped>

</style>