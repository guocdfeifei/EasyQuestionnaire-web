<template>
  <div id="manage">
    <nav-bar>
      <template v-slot:nav-center>
        <el-menu class="el-menu-demo" mode="horizontal" :default-active="navBarActive">
          <el-menu-item index="1">我的项目</el-menu-item>
          <el-menu-item index="2" @click="showTemplateContainer">常用模板</el-menu-item>
        </el-menu>
      </template>
      <template v-slot:nav-right>
        <div id="profileHead">
          <el-avatar :src="tempHeadIconLink" style="margin-top: 10px; float: right; margin-right: 50px"></el-avatar>
        </div>
      </template>
    </nav-bar>

    <el-main id="manage-main">
      <scroll-bar>
        <div id="boxcontainer">
          <div class="myProjectItemBox">
            <data-card card-name="新的项目..."
                       :card-style="getCardStyle(260)" :key="0"
                       class="myProjectItem">
              <template v-slot:cardBody>
                <div id="icon-wrap">
                  <i class="el-icon-plus" @click="gotoCreatequestionnaire"></i>
                </div>
              </template>
            </data-card>
            <data-card v-for="(questionnaire, index) in myQuestionnaire"
                       :key="questionnaire.questionnaireId"
                       :card-name="questionnaire.title"
                       :card-style="getCardStyle(260)"
                       class="myProjectItem">
              <template v-slot:cardHead>
                <el-tag size="mini" class="cardTags">问卷</el-tag>
              </template>
              <template v-slot:cardBody>
                <div id="card-body">
                  <el-link style="font-size: 9px; margin-left: 10px" :type="showConditionStyle(index)"
                           :underline="false">
                    {{showCondition(index)}}
                  </el-link>
                </div>
              </template>
              <template v-slot:cardFoot>
                <div id="card-foot">
                  <div class="card-icon-wrap">
                    <el-button icon="el-icon-edit" type="mini" class="card-bottom-button"
                               @click="gotoEdit(questionnaire.questionnaireId)">编辑
                    </el-button>
                  </div>
                  <div class="card-icon-wrap">
                    <el-button icon="el-icon-position" type="mini" class="card-bottom-button"
                               @click="gotoSpread(questionnaire.questionnaireId)">发布
                    </el-button>
                  </div>
                  <div class="card-icon-wrap">
                    <el-button icon="el-icon-document-copy" type="mini" class="card-bottom-button"
                               @click="gotoAlalysis(questionnaire.questionnaireId)">数据
                    </el-button>
                  </div>
                  <div class="card-icon-wrap">
                    <el-dropdown>
                      <el-button icon="el-icon-more-outline" type="mini" class="card-bottom-button"
                                 style="font-size: 14px;"></el-button>
                      <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item @click.native=gotoSpread(questionnaire.questionnaireId)>发布项目
                        </el-dropdown-item>
                        <el-dropdown-item @click.native="gotoPreview(questionnaire.questionnaireId)">预览项目
                        </el-dropdown-item>
                        <el-dropdown-item @click.native="confirmDelete(questionnaire.questionnaireId)">删除项目
                        </el-dropdown-item>
                      </el-dropdown-menu>
                    </el-dropdown>
                  </div>
                </div>
              </template>
            </data-card>
          </div>
        </div>
      </scroll-bar>
    </el-main>

    <!--问卷模板交互界面-->
    <el-dialog title="问卷模板(注意：所有数据均来源于爬虫 仅供学习交流使用)"
               :visible.sync="templatesConfig.templateContainerVisiable"
               width="75%" top="8vh" center>
      <template-list :template-data="this.templatesConfig.templateData"></template-list>
      <div id="templates-pagination">
        <el-pagination
                layout="prev, pager, next"
                :total="this.templatesConfig.totalPage * 10" :current-page.sync="templatesConfig.templateCurrentPage"
                @current-change="getNewPageData">
        </el-pagination>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  //公共组件
  import dataCard from "@/components/dataCard/dataCard";
  import scrollBar from "../../components/scrollBar/scrollBar";


  //数据处理
  import {deleteQuestionnaire, getAllQuestionnaire, getTemplates} from "../../network/questionnaire";
  import navBar from "../../components/navBar/navBar";
  import TemplateList from "./childComp/templateList";
  import {QuestionnaireCondition} from "../../models/questionnaire_model";

  export default {
    name: "manage",
    components: {
      TemplateList,
      scrollBar,
      navBar,
      dataCard,
    },
    created() {
      this.getQuesionNaire()
    },
    data() {
      return {
        tempHeadIconLink: "https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png",
        navBarActive: "1",
        myQuestionnaire: [],
        cardActive: [],
        templatesConfig: {
          templateContainerVisiable: false,
          templateData: [],
          templateCurrentPage: 1,
          totalPage: 0
        }
      }
    },
    methods: {
      gotoPreview(flag) {
        window.open(this.globalData.webBaseUrl + "/complete/" + flag + "?type=preview");
      },
      getNewPageData() {
        getTemplates(this.templatesConfig.templateCurrentPage)
          .then(res => {
            this.templatesConfig.templateData = res['data'];
          })
      },
      priviewTempate(flag) {
        console.log(flag);
      },
      showTemplateContainer() {
        this.getNewPageData();
        this.templatesConfig.templateContainerVisiable = true;
      },

      gotoAlalysis(flag) {
        this.$router.push('/analysis/' + flag);
      },

      gotoSpread(flag) {
        this.$router.push('/spread/' + flag);
      },

      getQuesionNaire() {
        getAllQuestionnaire()
          .then(res => {
            this.templatesConfig.totalPage = this.checkPage(res['tAmount']);
            for (let i = 0; i < res['questionnaires'].length; i++) {
              let q = new QuestionnaireCondition(res['questionnaires'][i]);
              this.myQuestionnaire.push(q);
            }
          })
          .catch(() => {
            this.$messageBox.showInfoMessage(this, "当前还没有问卷 快去创建一个吧");
          })
      },
      checkPage(tot) {
        return (!(tot % 10)) ? (tot / 10) : (tot / 10 + 1);
      },
      confirmDelete(qid) {
        this.$confirm('此操作将永久删除该项目, 是否继续?', '注意', {
          confirmButtonText: '确定删除',
          cancelButtonText: '我再想想',
          type: 'warning'
        }).then(() => {
          this.deleteQuesionNaire(qid)
        }).catch(() => {
          this.$messageBox.showInfoMessage(this, "用户已取消");
        });
      },

      deleteQuesionNaire(qid) {
        deleteQuestionnaire(qid)
          .then(() => {
            this.$messageBox.showSuccessMessage(this, "删除项目成功!");
            this.myQuestionnaire = [];
            this.getQuesionNaire();
          });
      },

      gotoEdit(target) {
        this.$router.push('/questionnaire/' + target);
      },

      showCondition(index) {
        let code = this.myQuestionnaire[index].condition;
        return code ? "发布中" : "未发布"
      },

      showConditionStyle(index) {
        let code = this.myQuestionnaire[index].condition;
        return code ? "success" : "warning"
      },

      getCardStyle(width) {
        return {
          "width": width + "px"
        }
      },

      gotoCreatequestionnaire() {
        this.$router.push('/questionnaire/new');
      }
    },
  }
</script>

<style scoped>


  #card-body {
    height: 50px;
  }

  #card-foot {
    display: flex;
  }

  #boxcontainer {
    display: flex;
    justify-content: center;
    height: 90vh;

  }

  .myProjectItemBox {
    display: flex;
    flex-wrap: wrap;
    width: 1200px;
    height: 200px;
    /*background-color: red;*/
  }

  .myProjectItem {
    padding: 20px;
  }

  .cardTags {
    padding-left: 12px;
    padding-right: 12px;
  }

  #manage-main {
    background-color: #f7f8fa;
    height: 100vh;
    overflow: hidden;
  }

  #icon-wrap {
    padding-left: 85px;
    padding-top: 10px;
    padding-bottom: 25px;
    font-size: 40px;
  }

  .card-icon-wrap {
    width: 80px;
  }

  .card-bottom-button {
    padding-left: 10px;
    padding-right: 10px;
    border: 0;
  }

  #manage {
    overflow: hidden;
    height: 100vh;
  }

  #templates-pagination {
    padding-top: 20px;
    display: flex;
    justify-content: center;
  }
</style>