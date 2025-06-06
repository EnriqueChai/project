<template>
  <div class="recomListContainer">
    <div class="list-content">
      <div class="recomLsitTop">
        <div class="paper-icon">
          <svg-icon icon-class="pdf" class="pdf-icon" />
        </div>
        <h3 @click="openModal" class="clickable-title">{{ paper.title || '未知标题' }}</h3>
      </div>

      <div class="recomLsitMid">
        <div class="authors">
          <span v-if="displayedAuthors.length > 0">
            <span v-for="(author, index) in displayedAuthors" :key="index">
              <span class="author-name">{{ author }}</span>
              <span v-if="index < displayedAuthors.length - 1">, </span>
            </span>
            <span v-if="hasEtAl"> et al.</span>
          </span>
          <span v-else>
            <span class="author-name">未知作者</span>
          </span>
        </div>

        <div class="paper-meta">
          <span class="meta-item">
            <i class="el-icon-date"></i> {{ formatYear(paper.time || paper.year) }}
          </span>
          <span class="meta-item">
            <i class="el-icon-reading"></i> 引用: {{ paper.citations !== undefined && paper.citations !== null ? paper.citations : 0 }}
          </span>
          <div class="theme-tags">
            <template v-if="paper.themes && paper.themes.length > 0">
              <el-tag 
                v-for="(theme, i) in paper.themes" 
                :key="i" 
                size="mini" 
                effect="plain" 
                type="info"
                class="theme-tag"
              >
                {{ theme }}
              </el-tag>
            </template>
            <el-tag 
              v-else
              size="mini" 
              effect="plain" 
              type="info"
              class="theme-tag"
            >
              未知主题
            </el-tag>
          </div>
          <el-button 
            size="mini" 
            type="primary" 
            plain 
            round 
            icon="el-icon-view"
            class="view-btn"
            @click.stop="openModal"
          >查看详情</el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PaperItem',
  props: {
    paper: {
      type: Object,
      required: true,
      default: () => ({
        title: '未知标题',
        authors: [],
        time: '',
        summary: '暂无摘要',
        citations: 0,
        pdfUrl: '',
        themes: []
      })
    }
  },
  data() {
    return {
      showModal: false,
      showModalContent: false,
      maxLength: 6 // 保持一致，最大显示作者数量
    };
  },
  computed: {
    displayedAuthors() {
      const authors = this.paper.authors || [];

      // 提取前 maxLength 个作者的名字
      return authors.slice(0, this.maxLength).map(author => {
        return this.formatAuthor(author);
      });
    },
    hasEtAl() {
      const authors = this.paper.authors || [];
      return authors.length > this.maxLength;
    }
  },
  methods: {
    formatAuthor(author) {
      if (!author) return '未知作者';
      return typeof author === 'object' ? (author.name || '未知作者') : author;
    },
    formatYear(timeStr) {
      if (!timeStr) return '未知日期';
      
      // 处理日期的多种可能情况
      const timeValue = String(timeStr);
      
      // 1. 如果是完整的日期字符串（如 "2023-01-15"）
      if (/^\d{4}-\d{2}-\d{2}$/.test(timeValue)) {
        return timeValue;
      }
      
      // 2. 如果是带时间的日期字符串（如 "2023-01-15 12:00:00"）
      if (/^\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2}$/.test(timeValue)) {
        return timeValue.split(' ')[0];
      }
      
      // 3. 如果是纯数字且是4位，直接作为年份
      if (/^\d{4}$/.test(timeValue)) return timeValue;
      
      // 4. 如果是year字段，直接获取
      if (this.paper && this.paper.year) {
        const year = String(this.paper.year);
        if (/^\d{4}$/.test(year)) return year;
      }
      
      // 5. 尝试从时间字符串中提取完整日期
      const dateMatch = timeValue.match(/\d{4}-\d{2}-\d{2}/);
      if (dateMatch) return dateMatch[0];
      
      // 6. 尝试匹配任何形式的4位数年份
      const yearMatch = timeValue.match(/\b(19|20)\d{2}\b/);
      if (yearMatch) return yearMatch[0];
      
      // 都无法匹配则返回未知日期
      return '未知日期';
    },
    openModal() {
      // 使用事件总线触发全局事件，由App.vue或Layout组件处理模态框显示
      this.$bus.$emit('showPaperModal', {
        paper: this.paper,
        show: true
      });
    },
    closeModal() {
      // 关闭的处理将由接收事件的组件处理
    },
    handleKeyDown(event) {
      // 键盘事件处理也将由处理模态框的组件负责
    },
    downloadPdf() {
      if (this.paper && this.paper.pdfUrl) {
        // 直接打开 PDF 链接
        window.open(this.paper.pdfUrl, '_blank');
      } else {
        this.$message({
          message: '暂无可下载的PDF文件',
          type: 'warning'
        });
      }
    }
  },
  mounted() {
    // 确保在组件挂载时数据已经准备好
    this.$nextTick(() => {
      if (this.paper) {
        // 手动触发一次数据处理，确保年份和引用量数据已准备就绪
        if (this.paper.time) {
          this.formatYear(this.paper.time);
        }
        if (this.paper.year) {
          this.formatYear(this.paper.year);
        }
      }
    });
  }
};
</script>

<style lang="scss">
.recomListContainer {
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 16px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  border-radius: 8px;
  position: relative;
  
  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  }
  
  .list-content {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .recomLsitTop {
    display: flex;
    align-items: flex-start;
    
    .paper-icon {
      margin-top: 4px;
      margin-right: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 28px;
      height: 28px;
      border-radius: 50%;
      background: linear-gradient(135deg, #ff9a9e 0%, #f56c6c 100%);
      box-shadow: 0 3px 8px rgba(245, 108, 108, 0.2);
      
      .pdf-icon {
        font-size: 16px;
        color: white;
      }
    }

    h3 {
      margin: 0;
      font-size: 18px;
      font-weight: 600;
      line-height: 1.4;
      color: #2c3e50;
      flex: 1;
      cursor: pointer;
      
      &:hover {
        color: #3498db;
      }
    }
  }

  .recomLsitMid {
    padding-left: 32px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    
    .authors {
      color: #7f8c8d;
      font-size: 14px;
      line-height: 1.5;
    }
    
    .author-name {
      color: #3498db;
      transition: color 0.2s;
      
      &:hover {
        color: #2980b9;
        text-decoration: underline;
      }
    }
    
    .paper-meta {
      display: flex;
      align-items: center;
      margin-top: 4px;
      flex-wrap: wrap;
      gap: 12px;
      
      .meta-item {
        display: flex;
        align-items: center;
        color: #95a5a6;
        font-size: 14px;
        
        i {
          margin-right: 4px;
          
          &.el-icon-date {
            color: #3498db;
          }
          
          &.el-icon-reading {
            color: #f39c12;
          }
        }
      }
      
      .theme-tags {
        display: flex;
        flex-wrap: wrap;
        gap: 6px;
        
        .theme-tag {
          font-size: 12px;
          height: 22px;
          line-height: 20px;
          padding: 0 6px;
          background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
          color: #2c3e50;
          border: none;
          
          &:nth-child(3n+1) {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
          }
          
          &:nth-child(3n+2) {
            background: linear-gradient(135deg, #fdfcfb 0%, #e2d1c3 100%);
          }
          
          &:nth-child(3n+3) {
            background: linear-gradient(135deg, #f5f7fa 0%, #b8c6db 100%);
          }
        }
      }
      
      .view-btn {
        margin-left: auto;
        transition: all 0.3s;
        background: linear-gradient(135deg, #00c6fb 0%, #005bea 100%);
        border: none;
        color: white;
        
        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 5px 12px rgba(0, 123, 255, 0.3);
          background: linear-gradient(135deg, #00b5e9 0%, #0052d6 100%);
        }
      }
    }
  }

  /* 模态框样式 */
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9999;
    backdrop-filter: blur(3px);
  }
  
  /* 模态框动画 */
  .modal-fade-enter-active {
    transition: all 0.3s ease-out;
  }
  
  .modal-fade-leave-active {
    transition: all 0.3s ease-in;
  }
  
  .modal-fade-enter-from,
  .modal-fade-leave-to {
    opacity: 0;
    transform: scale(0.9);
  }

  .modal-content {
    background-color: #fff;
    border-radius: 10px;
    width: 700px;
    max-width: 90%;
    position: relative;
    max-height: 80vh;
    display: flex;
    flex-direction: column;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
    overflow: hidden;
  }
  
  .modal-header {
    padding: 20px 30px;
    border-bottom: 1px solid #f0f2f5;
    background: linear-gradient(90deg, #11aacb 0%, #25a5fc 100%);
    
    h2 {
      margin: 0;
      font-size: 20px;
      font-weight: 600;
      color: white;
      line-height: 1.4;
    }
  }
  
  .modal-body {
    padding: 20px 30px;
    overflow-y: auto;
    flex: 1;
    
    .modal-section {
      margin-bottom: 24px;
      
      h3 {
        margin: 0 0 10px 0;
        font-size: 16px;
        color: #2c3e50;
        display: flex;
        align-items: center;
        
        i {
          margin-right: 6px;
          
          &.el-icon-user {
            color: #9b59b6;
          }
          
          &.el-icon-document-copy {
            color: #3498db;
          }
          
          &.el-icon-collection-tag {
            color: #f39c12;
          }
        }
      }
      
      p {
        margin: 0;
        line-height: 1.6;
      }
      
      &.theme-section {
        .themes-container {
          display: flex;
          flex-wrap: wrap;
          gap: 8px;
          margin-top: 10px;
          
          .large-theme-tag {
            font-size: 13px;
            padding: 6px 12px;
            border-radius: 4px;
            
            &:nth-child(5n+1) {
              background: linear-gradient(135deg, #a18cd1 0%, #fbc2eb 100%);
              border: none;
            }
            
            &:nth-child(5n+2) {
              background: linear-gradient(135deg, #84fab0 0%, #8fd3f4 100%);
              border: none;
            }
            
            &:nth-child(5n+3) {
              background: linear-gradient(135deg, #f6d365 0%, #fda085 100%);
              border: none;
            }
            
            &:nth-child(5n+4) {
              background: linear-gradient(135deg, #a1c4fd 0%, #c2e9fb 100%);
              border: none;
            }
            
            &:nth-child(5n+5) {
              background: linear-gradient(135deg, #cfd9df 0%, #e2ebf0 100%);
              border: none;
              color: #2c3e50;
            }
          }
        }
      }
      
      &.modal-info {
        display: flex;
        gap: 30px;
        
        .info-item {
          display: flex;
          align-items: flex-start;
          
          i {
            font-size: 18px;
            margin-right: 10px;
            margin-top: 3px;
            
            &.el-icon-date {
              color: #3498db;
            }
            
            &.el-icon-reading {
              color: #f39c12;
            }
          }
          
          h4 {
            margin: 0 0 4px 0;
            color: #7f8c8d;
            font-size: 14px;
            font-weight: normal;
          }
          
          p {
            margin: 0;
            font-weight: 500;
            color: #2c3e50;
          }
        }
      }
    }
    
    .author-list {
      line-height: 1.8;
    }
    
    .modal-author {
      color: #3498db;
      cursor: pointer;
      
      &:hover {
        text-decoration: underline;
      }
    }
    
    .paper-abstract {
      text-align: justify;
      color: #2c3e50;
      background-color: #f9f9f9;
      padding: 12px;
      border-radius: 6px;
      border-left: 3px solid #3498db;
    }
  }
  
  .modal-footer {
    padding: 15px 30px;
    border-top: 1px solid #f0f2f5;
    display: flex;
    justify-content: flex-end;
    gap: 12px;
    
    .el-button {
      transition: all 0.3s;
      
      &[type="primary"] {
        background: linear-gradient(135deg, #00c6fb 0%, #005bea 100%);
        border: none;
        
        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 5px 12px rgba(0, 123, 255, 0.3);
          background: linear-gradient(135deg, #00b5e9 0%, #0052d6 100%);
        }
      }
      
      &[type="success"] {
        background: linear-gradient(135deg, #84fab0 0%, #67c23a 100%);
        border: none;
        
        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 5px 12px rgba(103, 194, 58, 0.3);
          background: linear-gradient(135deg, #78efaa 0%, #5ab52e 100%);
        }
      }
    }
  }

  .close-button {
    position: absolute;
    top: 15px;
    right: 15px;
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: rgba(255, 255, 255, 0.8);
    width: 30px;
    height: 30px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
    z-index: 2;

    &:hover {
      background-color: rgba(255, 255, 255, 0.2);
      color: white;
    }
  }
}
</style>