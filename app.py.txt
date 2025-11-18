import streamlit as st
import pandas as pd
import base64
import time

# --- CẤU HÌNH TRANG ---
st.set_page_config(
    page_title="VeoMaster - Tool Tạo Video Đồng Nhất",
    page_icon="🎬",
    layout="wide",
    initial_sidebar_state="expanded"
)

# --- CSS TÙY CHỈNH CHO ĐẸP ---
st.markdown("""
<style>
    .main {background-color: #f0f2f6;}
    .stButton>button {width: 100%; border-radius: 10px; height: 3em; font-weight: bold;}
    .reportview-container .main .block-container{padding-top: 2rem;}
</style>
""", unsafe_allow_html=True)

# --- LOGIC XỬ LÝ PROMPT ---
def generate_prompt(char_name, char_desc, action, location, style, camera, lighting, seed_url):
    # Cấu trúc prompt tối ưu cho Veo/Runway/Sora
    prompt_structure = f"CINEMATIC VIDEO: {style} style. "
    
    # Subject (Quan trọng nhất)
    prompt_structure += f"CHARACTER: {char_desc} (referring to {char_name}). "
    
    # Action & Setting
    prompt_structure += f"ACTION: {action}. LOCATION: {location}. "
    
    # Tech Specs
    prompt_structure += f"CAMERA: {camera}. LIGHTING: {lighting}. 8k resolution, highly detailed."
    
    # Cú pháp Reference (Giả lập)
    if seed_url:
        prompt_structure += f" --cref {seed_url} --cw 90"
        
    return prompt_structure

# --- GIAO DIỆN (SIDEBAR) ---
with st.sidebar:
    st.image("https://cdn-icons-png.flaticon.com/512/3845/3845808.png", width=80)
    st.title("⚙️ CẤU HÌNH MASTER")
    st.info("Thiết lập nhân vật gốc tại đây để đảm bảo tính nhất quán.")
    
    st.subheader("1. Thông tin Nhân vật")
    char_name = st.text_input("Tên nhân vật", "Thầy Hoanh")
    char_desc = st.text_area("Mô tả ngoại hình (Cố định)", 
                             "A Vietnamese male teacher, 40 years old, kind face, wearing glasses and a white shirt, black hair", height=100)
    seed_url = st.text_input("Link ảnh gốc (Reference Image)", placeholder="https://imgur.com/...")
    
    st.subheader("2. Phong cách Chung")
    style_opt = st.selectbox("Phong cách nghệ thuật", 
                             ["Realistic Cinematic", "Pixar 3D Animation", "Anime Studio Ghibli", "Vintage 1990s TV", "Cyberpunk"])
    lighting_opt = st.selectbox("Ánh sáng", ["Natural Sunlight", "Cinematic Lighting", "Soft Morning Light", "Dark & Moody"])

# --- GIAO DIỆN CHÍNH ---
st.title("🎬 VEOMASTER: TRÌNH TẠO KỊCH BẢN VIDEO HÀNG LOẠT")
st.markdown("---")

st.write("### 📝 Bước 1: Nhập danh sách các cảnh quay")
st.caption("Hãy điền hành động và bối cảnh cho từng video bạn muốn tạo.")

# Dữ liệu mẫu
default_data = {
    "STT": ["1", "2", "3"],
    "Hành_Động (Action)": ["đang đi bộ vào lớp học và mỉm cười", "đang viết phấn lên bảng đen", "đang giảng bài say sưa cho học sinh"],
    "Bối_Cảnh (Location)": ["hành lang trường học nhiều nắng", "trong lớp học sáng sủa", "bục giảng lớp học"],
    "Góc_Máy (Camera)": ["Wide shot (Góc rộng)", "Close-up (Cận mặt)", "Medium shot (Trung cảnh)"]
}

df = pd.DataFrame(default_data)
edited_df = st.data_editor(df, num_rows="dynamic", use_container_width=True)

st.write("### 🚀 Bước 2: Xử lý và Xuất Prompts")

if st.button("⚡ TẠO DANH SÁCH PROMPTS NGAY"):
    with st.spinner('Đang viết kịch bản cho từng cảnh...'):
        time.sleep(1) # Giả lập thời gian xử lý
        
        results = []
        for idx, row in edited_df.iterrows():
            p = generate_prompt(
                char_name=char_name,
                char_desc=char_desc,
                action=row["Hành_Động (Action)"],
                location=row["Bối_Cảnh (Location)"],
                style=style_opt,
                camera=row["Góc_Máy (Camera)"],
                lighting=lighting_opt,
                seed_url=seed_url
            )
            results.append([row["STT"], p])
            
        result_df = pd.DataFrame(results, columns=["Cảnh", "Prompt Đã Tối Ưu (Copy dòng này vào Veo/Runway)"])
        
        st.success("Đã tạo xong! Hãy xem kết quả bên dưới.")
        st.dataframe(result_df, use_container_width=True)
        
        # Chức năng tải về
        csv = result_df.to_csv(index=False).encode('utf-8')
        st.download_button(
            label="📥 Tải file Excel (CSV) về máy",
            data=csv,
            file_name=f'Prompts_Video_{char_name}.csv',
            mime='text/csv',
        )
        
st.markdown("---")
st.markdown("Made for Creators | Powered by Python & Streamlit")