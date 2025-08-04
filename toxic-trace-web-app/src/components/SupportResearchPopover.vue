<template>
  <el-popover
    placement="bottom"
    :teleported="false"
    trigger="manual"
    :width="420"
    :offset="0"
    popper-class="funding-popover fancy-popover"
    :visible="visible"
  >
    <template #default>
      <div class="funding-content fancy-content">
        <div class="funding-header">
          <span class="funding-icon">
            <svg width="38" height="38" viewBox="0 0 38 38" fill="none">
              <circle cx="19" cy="19" r="19" fill="url(#purpleGradient)"/>
              <path d="M12 19c0-3.866 3.134-7 7-7s7 3.134 7 7c0 3.866-3.134 7-7 7s-7-3.134-7-7zm7-4a4 4 0 100 8 4 4 0 000-8z" fill="#fff"/>
              <defs>
                <linearGradient id="purpleGradient" x1="0" y1="0" x2="38" y2="38" gradientUnits="userSpaceOnUse">
                  <stop stop-color="#7b2cbf"/>
                  <stop offset="1" stop-color="#9d4edd"/>
                </linearGradient>
              </defs>
            </svg>
          </span>
          <h3 class="funding-title">Support Our Research</h3>
        </div>
        <div class="funding-links">
          <div class="funding-item">
            <a href="https://donate.stripe.com/8x214n9Pq834cUS30T1Nu04" 
               class="funding-button subscription-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🔬</span> Subscribe to BPA Research
            </a>
          </div>
          <div class="funding-item">
            <a href="https://buy.stripe.com/aFa9ATbXy6Z08EC1WP1Nu03" 
               class="funding-button support-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🧪</span> Support to Trace BPA in Food
            </a>
          </div>
          <div class="funding-item">
            <a href="https://donate.stripe.com/4gM5kD8Lmers9IGgRJ1Nu05" 
               class="funding-button subscription-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🧬</span> Subscribe to PFAS Research
            </a>
          </div>
          <div class="funding-item">
            <a href="https://buy.stripe.com/7sYdR92mY1EGbQO0SL1Nu06" 
               class="funding-button support-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🌱</span> Support to Trace PFAS in Food
            </a>
          </div>
          <div class="funding-item">
            <a href="https://donate.stripe.com/9B65kD5zaabc3kiatl1Nu08" 
               class="funding-button subscription-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🧫</span> Subscribe to Phthalates Research
            </a>
          </div>
          <div class="funding-item">
            <a href="https://buy.stripe.com/5kQeVd0eQcjk6wubxp1Nu07" 
               class="funding-button support-button fancy-button"
               target="_blank"
               rel="noopener noreferrer">
              <span class="button-icon">🦠</span> Support to Trace Phthalates in Food
            </a>
          </div>
        </div>
      </div>
    </template>
    <template #reference>
      <el-button 
        @click="togglePopover"
        type="danger"
      >
        <span class="popover-btn-icon">❤️</span> Support Research
      </el-button>
    </template>
  </el-popover>
</template>

<script>
export default {
  name: 'SupportResearchPopover',
  props: {
    visible: {
      type: Boolean,
      default: false
    }
  },
  emits: ['toggle'],
  methods: {
    togglePopover() {
      this.$emit('toggle');
    }
  }
};
</script>

<style scoped lang="scss">

:deep(.funding-popover.el-popover.el-popper),
.fancy-popover {
  padding: 0;
  background: rgba(255,255,255,0.85) !important;
  border: none;
  border-radius: 18px;
  box-shadow: 0 8px 32px rgba(123, 44, 191, 0.18), 0 1.5px 8px 0 rgba(0,0,0,0.08);
  backdrop-filter: blur(12px);
  z-index: 1000;
  overflow: hidden;
}

.fancy-content {
  padding: 32px 24px 24px 24px;
  border-radius: 18px;
  background: linear-gradient(135deg, #f3e6f9 0%, #e6ccf2 100%);
  box-shadow: 0 2px 12px rgba(123,44,191,0.08);
  position: relative;
}

.funding-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 18px;
}

.funding-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #7b2cbf 0%, #9d4edd 100%);
  border-radius: 50%;
  box-shadow: 0 2px 8px rgba(123,44,191,0.18);
  padding: 6px;
}

.funding-title {
  font-size: 22px;
  font-weight: 600;
  color: #7b2cbf;
  letter-spacing: 0.5px;
  text-shadow: 0 1px 8px #e6ccf2;
}

.funding-links {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.funding-item {
  display: flex;
  justify-content: center;
}

.fancy-button {
  display: flex;
  align-items: center;
  gap: 10px;
  color: white;
  padding: 14px 28px;
  text-align: center;
  text-decoration: none;
  font-size: 16px;
  font-weight: 500;
  border-radius: 10px;
  border: none;
  box-shadow: 0 2px 12px rgba(123,44,191,0.12);
  background: linear-gradient(135deg, #7b2cbf 0%, #9d4edd 100%);
  transition: all 0.25s cubic-bezier(.4,2,.3,1);
  min-width: 220px;
  position: relative;
  overflow: hidden;
}

.fancy-button .button-icon {
  font-size: 20px;
  filter: drop-shadow(0 1px 2px #e6ccf2);
}

.fancy-button:hover {
  background: linear-gradient(135deg, #9d4edd 0%, #7b2cbf 100%);
  transform: translateY(-2px) scale(1.04);
  box-shadow: 0 6px 24px rgba(123,44,191,0.18);
}

.subscription-button {
  background: linear-gradient(135deg, #003366 0%, #7b2cbf 100%);
}
.subscription-button:hover {
  background: linear-gradient(135deg, #004488 0%, #9d4edd 100%);
}
.support-button {
  background: linear-gradient(135deg, #0B3D91 0%, #9d4edd 100%);
}
.support-button:hover {
  background: linear-gradient(135deg, #0D4BA3 0%, #7b2cbf 100%);
}

.fancy-popover-btn {
  font-size: 16px !important;
  font-weight: 500;
  border-radius: 12px !important;
  background: linear-gradient(135deg, #7b2cbf 0%, #9d4edd 100%) !important;
  color: white !important;
  box-shadow: 0 2px 8px rgba(123,44,191,0.18);
  padding: 10px 24px !important;
  transition: all 0.25s cubic-bezier(.4,2,.3,1);
  display: flex;
  align-items: center;
  gap: 10px;
}
.fancy-popover-btn-icon {
  font-size: 22px;
  filter: drop-shadow(0 1px 2px #e6ccf2);
}
.fancy-popover-btn-text {
  font-size: 16px;
  font-weight: 500;
}
.fancy-popover-btn:hover {
  background: linear-gradient(135deg, #9d4edd 0%, #7b2cbf 100%) !important;
  transform: scale(1.05);
  box-shadow: 0 6px 24px rgba(123,44,191,0.18);
}
</style>
