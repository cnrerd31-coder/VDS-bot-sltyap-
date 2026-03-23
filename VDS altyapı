# ⛓️━━━━━━━━━━━━⛓️
# 👑 KOD SAHİBİ » @luna carma siker atar 🎖️
# 🔥 DAHA FAZLASI » @luan carma siker atar 🇹🇷
# ⛓️━━━━━━━━━━━━⛓️

# -*- coding: utf-8 -*-
import telebot
import subprocess
import os
import zipfile
import tempfile
import shutil
from telebot import types
import time
from datetime import datetime, timedelta
import sqlite3
import logging
import threading
import sys
import atexit
import random

# ================================
# 🌟 KONFİGÜRASYON - RENKLİ EMOJİLER 🌟
# ================================
TOKEN = '8668348358:AAH6J2URnN5wW_vpcNYQ-t-Eb5vTq08P50I' 
OWNER_ID = 7250471858
ADMIN_ID = 7250471858

# 🌟 GERÇEK KANAL DOĞRULAMA SİSTEMİ 🌟
REQUIRED_CHANNELS = [
    {
        'id': -1003827548507
,  # Sezy Tool kanal ID
        'name': '🚀 gleary sex Kanalı',
        'url': 'https://t.me/+stWHqpc0rY1hN2Q0',
        'username': 'GLEARY ARSİV'
    },
    {
        'id': -1003750786357,  # Chat Sunucu kanal ID
        'name': '💬 Chat Sunucusu',
        'url': 'https://t.me/+S_FUeFcdjvJhZDlk',
        'username': 'GLEARY CHAT'
    }
]

# 🌟 DOSYA YAPILARI 🌟
BASE_DIR = os.path.abspath(os.path.dirname(__file__))
UPLOAD_BOTS_DIR = os.path.join(BASE_DIR, 'upload_bots')
IROTECH_DIR = os.path.join(BASE_DIR, 'inf')
DATABASE_PATH = os.path.join(IROTECH_DIR, 'bot_data.db')

# Klasörleri oluştur
os.makedirs(UPLOAD_BOTS_DIR, exist_ok=True)
os.makedirs(IROTECH_DIR, exist_ok=True)

# Bot'u başlat
bot = telebot.TeleBot(TOKEN, parse_mode='HTML')

# ================================
# 🌟 RENKLİ LOG SİSTEMİ 🌟
# ================================
class ColorFormatter(logging.Formatter):
    COLORS = {
        'DEBUG': '\033[94m',      # Mavi
        'INFO': '\033[92m',       # Yeşil
        'WARNING': '\033[93m',    # Sarı
        'ERROR': '\033[91m',      # Kırmızı
        'CRITICAL': '\033[95m',   # Mor
        'RESET': '\033[0m'        # Sıfırla
    }
    
    def format(self, record):
        log_color = self.COLORS.get(record.levelname, self.COLORS['RESET'])
        message = super().format(record)
        return f"{log_color}{message}{self.COLORS['RESET']}"

# Logging setup
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Konsol handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_formatter = ColorFormatter('✨ %(asctime)s - %(levelname)s - %(message)s')
console_handler.setFormatter(console_formatter)

# Dosya handler
file_handler = logging.FileHandler('bot_activity.log', encoding='utf-8')
file_handler.setLevel(logging.INFO)
file_formatter = logging.Formatter('📝 %(asctime)s - %(levelname)s - %(message)s')
file_handler.setFormatter(file_formatter)

logger.addHandler(console_handler)
logger.addHandler(file_handler)

# ================================
# 🌟 VERİ YAPILARI 🌟
# ================================
bot_scripts = {}
user_files = {}
active_users = set()
admin_ids = {ADMIN_ID, OWNER_ID}
pending_approvals = {}
verified_users = set()
user_states = {}  # Kullanıcı durumlarını takip et

# ================================
# 🌟 VERİTABANI SİSTEMİ 🌟
# ================================
def init_db():
    """🎯 Veritabanını başlat"""
    logger.info("📊 Veritabanı başlatılıyor...")
    try:
        conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
        c = conn.cursor()
        
        # Kullanıcı dosyaları tablosu
        c.execute('''CREATE TABLE IF NOT EXISTS user_files
                     (user_id INTEGER, 
                      file_name TEXT, 
                      file_type TEXT, 
                      status TEXT DEFAULT 'pending',
                      upload_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                      PRIMARY KEY (user_id, file_name))''')
        
        # Aktif kullanıcılar tablosu
        c.execute('''CREATE TABLE IF NOT EXISTS active_users
                     (user_id INTEGER PRIMARY KEY,
                      join_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                      last_active TIMESTAMP DEFAULT CURRENT_TIMESTAMP)''')
        
        # Adminler tablosu
        c.execute('''CREATE TABLE IF NOT EXISTS admins
                     (user_id INTEGER PRIMARY KEY,
                      role TEXT DEFAULT 'admin')''')
        
        # Doğrulanmış kullanıcılar tablosu
        c.execute('''CREATE TABLE IF NOT EXISTS verified_users
                     (user_id INTEGER PRIMARY KEY,
                      verified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                      verification_count INTEGER DEFAULT 1)''')
        
        # Bot istatistikleri tablosu
        c.execute('''CREATE TABLE IF NOT EXISTS bot_stats
                     (stat_key TEXT PRIMARY KEY,
                      stat_value TEXT)''')
        
        # Sahip ve admin ekle
        c.execute('INSERT OR IGNORE INTO admins (user_id, role) VALUES (?, ?)', 
                  (OWNER_ID, 'owner'))
        if ADMIN_ID != OWNER_ID:
            c.execute('INSERT OR IGNORE INTO admins (user_id) VALUES (?)', (ADMIN_ID,))
        
        conn.commit()
        conn.close()
        logger.info("✅ Veritabanı başarıyla başlatıldı!")
    except Exception as e:
        logger.error(f"❌ Veritabanı hatası: {e}", exc_info=True)

def load_data():
    """🎯 Verileri veritabanından yükle"""
    logger.info("📥 Veriler yükleniyor...")
    try:
        conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
        c = conn.cursor()

        # Kullanıcı dosyalarını yükle
        c.execute('SELECT user_id, file_name, file_type, status FROM user_files')
        for user_id, file_name, file_type, status in c.fetchall():
            if user_id not in user_files:
                user_files[user_id] = []
            user_files[user_id].append((file_name, file_type, status))

        # Aktif kullanıcıları yükle
        c.execute('SELECT user_id FROM active_users')
        active_users.update(user_id for (user_id,) in c.fetchall())

        # Adminleri yükle
        c.execute('SELECT user_id FROM admins')
        admin_ids.update(user_id for (user_id,) in c.fetchall())

        # Doğrulanmış kullanıcıları yükle
        c.execute('SELECT user_id FROM verified_users')
        verified_users.update(user_id for (user_id,) in c.fetchall())

        conn.close()
        logger.info(f"✅ {len(active_users)} aktif kullanıcı yüklendi!")
    except Exception as e:
        logger.error(f"❌ Veri yükleme hatası: {e}", exc_info=True)

# Veritabanını başlat
init_db()
load_data()

# ================================
# 🌟 YARDIMCI FONKSİYONLAR 🌟
# ================================
def get_user_folder(user_id):
    """📂 Kullanıcı klasörünü oluştur"""
    user_folder = os.path.join(UPLOAD_BOTS_DIR, str(user_id))
    os.makedirs(user_folder, exist_ok=True)
    return user_folder

def get_user_file_count(user_id):
    """🔢 Kullanıcının dosya sayısını al"""
    return len(user_files.get(user_id, []))

def is_bot_running(script_owner_id, file_name):
    """⚙️ Bot'un çalışıp çalışmadığını kontrol et"""
    script_key = f"{script_owner_id}_{file_name}"
    script_info = bot_scripts.get(script_key)
    if script_info and script_info.get('process'):
        try:
            proc = script_info['process']
            if hasattr(proc, 'poll'):
                return proc.poll() is None
            return True
        except Exception as e:
            logger.error(f"❌ Process kontrol hatası: {e}")
            return False
    return False

def check_channel_membership(user_id):
    """🔍 Kullanıcının kanallara üye olup olmadığını kontrol et"""
    try:
        for channel in REQUIRED_CHANNELS:
            try:
                member = bot.get_chat_member(channel['id'], user_id)
                if member.status in ['left', 'kicked', 'banned']:
                    return False, channel['name']
            except Exception as e:
                logger.error(f"❌ Kanal kontrol hatası {channel['name']}: {e}")
                return False, channel['name']
        
        # DÜZELTME: return satırı 'for' döngüsünün DIŞINDA 
        # ama 'try' bloğunun İÇİNDE (en sonda) olmalı.
        return True, None 

    except Exception as e:
        logger.error(f"❌ Genel kanal kontrol hatası: {e}")
        return False, "Bilinmeyen Kanal"

def save_user_file(user_id, file_name, file_type, status='pending'):
    """💾 Kullanıcı dosyasını veritabanına kaydet"""
    try:
        conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
        c = conn.cursor()
        c.execute('INSERT OR REPLACE INTO user_files (user_id, file_name, file_type, status) VALUES (?, ?, ?, ?)',
                  (user_id, file_name, file_type, status))
        conn.commit()
        conn.close()
        
        # Bellekte güncelle
        if user_id not in user_files:
            user_files[user_id] = []
        user_files[user_id] = [(fn, ft, st) for fn, ft, st in user_files[user_id] if fn != file_name]
        user_files[user_id].append((file_name, file_type, status))
        
        return True
    except Exception as e:
        logger.error(f"❌ Dosya kaydetme hatası: {e}")
        return False

# ================================
# 🌟 GÜZEL EMOJİLİ KLAVYELER 🌟
# ================================
def create_main_keyboard(user_id):
    """🎹 Ana menü klavyesi"""
    keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    
    # Rastgele emoji kombinasyonları
    emoji_sets = [
        ['🚀 YÜKLE', '📁 DOSYALARIM', '⚡ HIZ', '📊 İSTATİSTİK'],
        ['📤 GÖNDER', '🗂️ ARŞİV', '💨 TEST', '📈 ANALİZ'],
        ['✨ YÜKLE', '🌟 DOSYALAR', '⚡ TEST', '📊 VERİ']
    ]
    
    emoji_choice = random.choice(emoji_sets)
    
    if user_id in admin_ids:
        buttons = [
            f'{emoji_choice[0]}',
            f'{emoji_choice[1]}',
            f'{emoji_choice[2]}',
            f'{emoji_choice[3]}',
            '👑 ONAYLAR',
            '🔄 YENİLE'
        ]
        keyboard.row(buttons[0], buttons[1])
        keyboard.row(buttons[2], buttons[3])
        keyboard.row(buttons[4], buttons[5])
    else:
        buttons = [
            f'{emoji_choice[0]}',
            f'{emoji_choice[1]}',
            f'{emoji_choice[2]}',
            f'{emoji_choice[3]}',
            '🔄 YENİLE'
        ]
        keyboard.row(buttons[0], buttons[1])
        keyboard.row(buttons[2], buttons[3])
        keyboard.row(buttons[4])
    
    return keyboard

def create_files_keyboard(user_id):
    """🗂️ Dosya listesi klavyesi"""
    keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    user_files_list = user_files.get(user_id, [])
    
    emoji_status = {
        'approved': ['✅', '🎯', '✨', '🌟'],
        'pending': ['⏳', '🕒', '⏱️', '⌛'],
        'rejected': ['❌', '🚫', '⛔', '💔']
    }
    
    buttons = []
    for file_name, file_type, status in user_files_list[:8]:  # Max 8 dosya göster
        if status == 'approved':
            is_running = is_bot_running(user_id, file_name)
            if is_running:
                status_emoji = random.choice(['🚀', '⚡', '💫', '🔥'])
            else:
                status_emoji = random.choice(['⏸️', '🛑', '⏯️', '⏹️'])
        else:
            status_emoji = random.choice(emoji_status.get(status, ['❓']))
        
        # Dosya adını kısalt
        display_name = file_name[:15] + '...' if len(file_name) > 15 else file_name
        btn_text = f"{status_emoji} {display_name}"
        buttons.append(btn_text)
    
    # Dosya butonlarını ekle
    for i in range(0, len(buttons), 2):
        if i+1 < len(buttons):
            keyboard.row(buttons[i], buttons[i+1])
        else:
            keyboard.row(buttons[i])
    
    # Geri butonu
    keyboard.row('🏠 ANA MENÜ')
    
    return keyboard

def create_channel_check_inline_keyboard():
    """🔗 Kanal kontrol inline klavyesi"""
    keyboard = types.InlineKeyboardMarkup(row_width=1)
    
    emoji_channels = ['🚀', '💬', '📢', '🌟', '✨']
    
    for idx, channel in enumerate(REQUIRED_CHANNELS):
        emoji = emoji_channels[idx % len(emoji_channels)]
        keyboard.add(
            types.InlineKeyboardButton(
                text=f"{emoji} {channel['name']}",
                url=channel['url']
            )
        )
    
    keyboard.add(
        types.InlineKeyboardButton(
            text="✅ KONTROL ET",
            callback_data="check_channels"
        )
    )
    
    return keyboard

def create_file_control_keyboard(file_name, status, is_running):
    """🎮 Dosya kontrol klavyesi"""
    keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    
    if status == 'approved':
        if is_running:
            buttons = [
                '⏸️ DURDUR',
                '🔄 YENİDEN',
                '🗑️ SİL',
                '📋 LOGLAR'
            ]
            keyboard.row(buttons[0], buttons[1])
            keyboard.row(buttons[2], buttons[3])
        else:
            buttons = [
                '🚀 BAŞLAT',
                '🗑️ SİL',
                '🔙 GERİ'
            ]
            keyboard.row(buttons[0], buttons[1])
            keyboard.row(buttons[2])
    else:
        keyboard.row('🔙 GERİ')
    
    keyboard.row('🏠 ANA MENÜ')
    return keyboard

def create_approval_inline_keyboard(file_id):
    """👑 Admin onay klavyesi"""
    keyboard = types.InlineKeyboardMarkup(row_width=2)
    keyboard.add(
        types.InlineKeyboardButton("✅ ONAYLA", callback_data=f"approve_{file_id}"),
        types.InlineKeyboardButton("❌ REDDET", callback_data=f"reject_{file_id}")
    )
    return keyboard

# ================================
# 🌟 BOT ÇALIŞTIRMA SİSTEMİ 🌟
# ================================
def run_bot_with_log(user_id, file_name, file_path, file_type):
    """⚙️ Bot'u log ile çalıştır"""
    def target():
        try:
            script_key = f"{user_id}_{file_name}"
            
            # Önceki process'i temizle
            if script_key in bot_scripts:
                old_proc = bot_scripts[script_key].get('process')
                if old_proc and old_proc.poll() is None:
                    old_proc.terminate()
                    time.sleep(1)
            
            if file_type == 'py':
                proc = subprocess.Popen(
                    [sys.executable, file_path],
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE,
                    text=True,
                    bufsize=1,
                    universal_newlines=True
                )
            elif file_type == 'js':
                proc = subprocess.Popen(
                    ['node', file_path],
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE,
                    text=True,
                    bufsize=1,
                    universal_newlines=True
                )
            else:
                logger.error(f"❌ Bilinmeyen dosya tipi: {file_type}")
                return
            
            bot_scripts[script_key] = {
                'process': proc,
                'file_name': file_name,
                'user_id': user_id,
                'file_type': file_type,
                'start_time': datetime.now(),
                'log_file': f"logs/{user_id}_{file_name}.log"
            }
            
            logger.info(f"🚀 Bot başlatıldı: {script_key}")
            
            # Logları kaydet
            def read_output(proc, script_key):
                try:
                    while True:
                        line = proc.stdout.readline()
                        if line:
                            clean_line = line.strip()
                            if clean_line:
                                logger.info(f"[{script_key}] {clean_line}")
                        elif proc.poll() is not None:
                            break
                except Exception as e:
                    logger.error(f"❌ Log okuma hatası: {e}")
                finally:
                    if script_key in bot_scripts:
                        del bot_scripts[script_key]
            
            threading.Thread(target=read_output, args=(proc, script_key), daemon=True).start()
            
        except Exception as e:
            logger.error(f"❌ Bot başlatma hatası: {e}", exc_info=True)
            bot.send_message(user_id, f"❌ Bot başlatılamadı: {str(e)[:200]}")

    threading.Thread(target=target, daemon=True).start()

# ================================
# 🌟 KANAL DOĞRULAMA SİSTEMİ 🌟
# ================================
@bot.callback_query_handler(func=lambda call: call.data == "check_channels")
def check_channels_callback(call):
    """🔍 Kanal kontrol callback'i"""
    user_id = call.from_user.id
    user_name = call.from_user.first_name
    
    try:
        bot.answer_callback_query(call.id, "🔍 Kontrol ediliyor...")
        
        # Kanal kontrolü yap
        is_member, channel_name = check_channel_membership(user_id)
        
        if is_member:
            # Kullanıcıyı doğrula
            try:
                conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
                c = conn.cursor()
                verified_at = datetime.now().isoformat()
                c.execute('INSERT OR REPLACE INTO verified_users (user_id, verified_at) VALUES (?, ?)', 
                         (user_id, verified_at))
                
                # Aktif kullanıcılara ekle
                c.execute('INSERT OR IGNORE INTO active_users (user_id) VALUES (?)', (user_id,))
                conn.commit()
                conn.close()
                
                verified_users.add(user_id)
                active_users.add(user_id)
                logger.info(f"✅ Kullanıcı doğrulandı: {user_id} ({user_name})")
                
                # Mesajı güncelle
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"🎉 **TEBRİKLER {user_name}!**\n\n✅ Tüm kanallara üyeliğiniz doğrulandı!\n\nArtık botu tam olarak kullanabilirsiniz! 🚀",
                    parse_mode='Markdown',
                    reply_markup=None
                )
                
                # Hoş geldin mesajı gönder
                welcome_messages = [
                    f"🌟 **Hoş Geldin {user_name}!**\n\nBotumuza katıldığınız için teşekkürler! 🎉\n\nArtık botlarınızı yükleyip yönetebilirsiniz!",
                    f"✨ **Merhaba {user_name}!**\n\nBaşarıyla doğrulandınız! 🎊\n\nHadi ilk botunuzu yükleyelim! 🚀",
                    f"🎯 **Hoş Geldin {user_name}!**\n\nDoğrulama tamamlandı! ✅\n\nŞimdi harika botlar oluşturma zamanı! 💫"
                ]
                
                bot.send_message(
                    user_id,
                    random.choice(welcome_messages),
                    parse_mode='Markdown',
                    reply_markup=create_main_keyboard(user_id)
                )
                
            except Exception as e:
                logger.error(f"❌ Doğrulama kaydetme hatası: {e}")
                bot.answer_callback_query(call.id, "❌ Bir hata oluştu!", show_alert=True)
        else:
            # Hangi kanala katılmadığını belirt
            bot.answer_callback_query(
                call.id, 
                f"❌ {channel_name} kanalına katılmamışsınız!",
                show_alert=True
            )
            
    except Exception as e:
        logger.error(f"❌ Kanal kontrol hatası: {e}")
        bot.answer_callback_query(call.id, "❌ Kontrol sırasında hata!", show_alert=True)

# ================================
# 🌟 START KOMUTU - SÜPER GÖRSELLİ 🌟
# ================================
@bot.message_handler(commands=['start', 'help', 'menu'])
def command_send_welcome(message):
    user_id = message.from_user.id
    user_name = message.from_user.first_name
    
    logger.info(f"🚀 Başlangıç isteği: {user_id} ({user_name})")
    
    # Admin doğrudan geçsin
    if user_id in admin_ids:
        show_main_menu(message)
        return
    
    # Zaten doğrulanmışsa ana menüyü göster
    if user_id in verified_users:
        show_main_menu(message)
        return
    
    # GÜZEL KANAL DOĞRULAMA EKRANI
    welcome_messages = [
        f"✨ **MERHABA {user_name}!** ✨\n\n🎉 Botumuza hoş geldin!\n\n🚀 Tüm özellikleri kullanabilmek için aşağıdaki kanallara katılman gerekiyor:",
        f"🌟 **HOŞ GELDİN {user_name}!** 🌟\n\n🔥 Harika botlar oluşturmaya hazır mısın?\n\nÖnce bu harika topluluğa katılmalısın:",
        f"🎯 **MERHABA {user_name}!** 🎯\n\n💫 Bot geliştirme macerana başlamak için\n\naşağıdaki kanallarımıza katılman yeterli:"
    ]
    
    channel_list = ""
    emoji_list = ["🚀", "💬", "🌟", "✨", "🔥", "🎯"]
    for idx, channel in enumerate(REQUIRED_CHANNELS):
        emoji = emoji_list[idx % len(emoji_list)]
        channel_list += f"\n{emoji} {channel['name']}"
    
    full_message = f"{random.choice(welcome_messages)}\n\n{channel_list}\n\n✅ Kanallara katıldıktan sonra 'KONTROL ET' butonuna bas!"
    
    try:
        bot.send_message(
            user_id,
            full_message,
            parse_mode='Markdown',
            reply_markup=create_channel_check_inline_keyboard()
        )
    except Exception as e:
        logger.error(f"❌ Başlangıç mesajı hatası: {e}")
        bot.send_message(user_id, "✨ Hoş geldin! Lütfen kanallarımıza katılın.")

def show_main_menu(message):
    """🎯 Ana menüyü göster"""
    user_id = message.from_user.id
    user_name = message.from_user.first_name
    
    # Aktif kullanıcı olarak kaydet
    if user_id not in active_users:
        try:
            conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
            c = conn.cursor()
            c.execute('INSERT OR IGNORE INTO active_users (user_id) VALUES (?)', (user_id,))
            c.execute('UPDATE active_users SET last_active = CURRENT_TIMESTAMP WHERE user_id = ?', (user_id,))
            conn.commit()
            conn.close()
            active_users.add(user_id)
        except Exception as e:
            logger.error(f"❌ Aktif kullanıcı kayıt hatası: {e}")
    
    current_files = get_user_file_count(user_id)
    
    # Kullanıcı statüsü
    if user_id == OWNER_ID:
        user_status = "👑 SAHİP"
        status_emoji = "👑"
    elif user_id in admin_ids:
        user_status = "🔧 ADMIN"
        status_emoji = "🔧"
    else:
        user_status = "👤 KULLANICI"
        status_emoji = "⭐"
    
    # Rastgele hoş geldin mesajları
    welcome_templates = [
        f"{status_emoji} **ANA MENÜ** {status_emoji}\n\n👤 **{user_name}**\n🎯 Seviye: {user_status}\n📂 Dosyalar: {current_files}/∞\n\n✨ Aşağıdaki butonlarla harika işler yap!",
        f"🌟 **HOŞ GELDİN {user_name}!** 🌟\n\n📊 Durum: {user_status}\n🗂️ Aktif Dosya: {current_files}\n\n🚀 Hadi bir şeyler yapalım!",
        f"🎯 **KONTROL PANELİ** 🎯\n\n👋 Merhaba {user_name}!\n📈 Statü: {user_status}\n📁 Yüklenen: {current_files} dosya\n\n💫 Ne yapmak istersin?"
    ]
    
    try:
        bot.send_message(
            user_id,
            random.choice(welcome_templates),
            parse_mode='Markdown',
            reply_markup=create_main_keyboard(user_id)
        )
    except Exception as e:
        logger.error(f"❌ Ana menü gönderme hatası: {e}")

# ================================
# 🌟 CALLBACK HANDLER 🌟
# ================================
@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    user_id = call.from_user.id
    data = call.data
    
    logger.info(f"🔔 Callback: {user_id} -> {data}")
    
    # Admin onay/reddetme
    if data.startswith("approve_") or data.startswith("reject_"):
        if user_id not in admin_ids:
            bot.answer_callback_query(call.id, "❌ Yetkiniz yok!", show_alert=True)
            return
        
        action, file_id = data.split("_", 1)
        
        if file_id not in pending_approvals:
            bot.answer_callback_query(call.id, "✅ Zaten işlendi!", show_alert=True)
            return
        
        file_info = pending_approvals[file_id]
        target_user_id = file_info['user_id']
        file_name = file_info['file_name']
        file_type = file_info['file_type']
        
        if action == "approve":
            # Dosyayı onayla
            if save_user_file(target_user_id, file_name, file_type, 'approved'):
                # Kullanıcıya bildir
                try:
                    bot.send_message(
                        target_user_id,
                        f"🎉 **DOSYAN ONAYLANDI!**\n\n✅ `{file_name}`\n\nArtık botunu başlatabilirsin! 🚀",
                        parse_mode='Markdown'
                    )
                except:
                    pass
                
                # Admin mesajını güncelle
                try:
                    bot.edit_message_caption(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        caption=f"✅ **ONAYLANDI**\n👤 {file_info['user_name']}\n📄 {file_name}\n🎯 {file_type.upper()}",
                        reply_markup=None
                    )
                except:
                    bot.edit_message_text(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        text=f"✅ **ONAYLANDI**\n👤 {file_info['user_name']}\n📄 {file_name}\n🎯 {file_type.upper()}",
                        reply_markup=None
                    )
                
                bot.answer_callback_query(call.id, "✅ Onaylandı!")
            
        elif action == "reject":
            # Dosyayı reddet ve sil
            user_folder = get_user_folder(target_user_id)
            file_path = os.path.join(user_folder, file_name)
            
            try:
                if os.path.exists(file_path):
                    os.remove(file_path)
            except:
                pass
            
            # Veritabanından sil
            try:
                conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
                c = conn.cursor()
                c.execute('DELETE FROM user_files WHERE user_id=? AND file_name=?', 
                         (target_user_id, file_name))
                conn.commit()
                conn.close()
                
                # Bellekten sil
                if target_user_id in user_files:
                    user_files[target_user_id] = [
                        (fn, ft, st) for fn, ft, st in user_files[target_user_id] 
                        if fn != file_name
                    ]
                
                # Kullanıcıya bildir
                try:
                    bot.send_message(
                        target_user_id,
                        f"❌ **DOSYAN REDDEDİLDİ**\n\n`{file_name}`\n\nDosyan admin tarafından reddedildi.",
                        parse_mode='Markdown'
                    )
                except:
                    pass
                
                # Admin mesajını güncelle
                try:
                    bot.edit_message_caption(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        caption=f"❌ **REDDEDİLDİ**\n👤 {file_info['user_name']}\n📄 {file_name}",
                        reply_markup=None
                    )
                except:
                    bot.edit_message_text(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        text=f"❌ **REDDEDİLDİ**\n👤 {file_info['user_name']}\n📄 {file_name}",
                        reply_markup=None
                    )
                
                bot.answer_callback_query(call.id, "❌ Reddedildi!")
                
            except Exception as e:
                logger.error(f"❌ Reddetme hatası: {e}")
                bot.answer_callback_query(call.id, "❌ Hata oluştu!", show_alert=True)
        
        # Bekleyenlerden kaldır
        if file_id in pending_approvals:
            del pending_approvals[file_id]

# ================================
# 🌟 MESAJ HANDLER'LARI 🌟
# ================================
@bot.message_handler(func=lambda message: True)
def handle_messages(message):
    user_id = message.from_user.id
    text = message.text
    
    if not text:
        return
    
    logger.info(f"💬 Mesaj: {user_id} -> {text}")
    
    # Doğrulama kontrolü
    if user_id not in verified_users and user_id not in admin_ids:
        command_send_welcome(message)
        return
    
    # Dosya Yükle
    if 'YÜKLE' in text or 'GÖNDER' in text:
        handle_upload(message)
    
    # Dosyalarım
    elif 'DOSYALAR' in text or 'ARŞİV' in text:
        handle_my_files(message)
    
    # Hız Testi
    elif 'HIZ' in text or 'TEST' in text:
        handle_speed(message)
    
    # İstatistik
    elif 'İSTATİSTİK' in text or 'ANALİZ' in text or 'VERİ' in text:
        handle_stats(message)
    
    # Admin Onaylar
    elif 'ONAYLAR' in text and user_id in admin_ids:
        handle_pending_approvals(message)
    
    # Yenile
    elif 'YENİLE' in text:
        show_main_menu(message)
    
    # Ana Menü
    elif 'ANA MENÜ' in text or 'GERİ' in text:
        show_main_menu(message)
    
    # Dosya Kontrol İşlemleri
    elif text in ['🚀 BAŞLAT', '⏸️ DURDUR', '🔄 YENİDEN', '🗑️ SİL', '📋 LOGLAR']:
        handle_file_action(message, text)
    
    # Dosya butonları (emoji + isim)
    else:
        # Dosya butonu mu kontrol et
        user_files_list = user_files.get(user_id, [])
        for file_name, file_type, status in user_files_list:
            display_name = file_name[:15] + '...' if len(file_name) > 15 else file_name
            possible_buttons = [
                f"✅ {display_name}", f"🎯 {display_name}", f"✨ {display_name}", f"🌟 {display_name}",
                f"⏳ {display_name}", f"🕒 {display_name}", f"⏱️ {display_name}",
                f"❌ {display_name}", f"🚫 {display_name}",
                f"🚀 {display_name}", f"⚡ {display_name}", f"💫 {display_name}", f"🔥 {display_name}",
                f"⏸️ {display_name}", f"🛑 {display_name}"
            ]
            
            if text in possible_buttons:
                handle_file_control(message, user_id, file_name, file_type, status)
                return
        
        # Bilinmeyen komut
        bot.send_message(
            user_id,
            "❌ Anlamadım!\n\nLütfen butonları kullan veya /start yaz.",
            reply_markup=create_main_keyboard(user_id)
        )

def handle_upload(message):
    """📤 Dosya yükleme"""
    user_id = message.from_user.id
    
    # Limit kontrolü
    current_files = get_user_file_count(user_id)
    if current_files >= 20:
        bot.send_message(
            user_id,
            f"❌ **LİMİT DOLDU!**\n\n📊 Mevcut: {current_files}/20\n\n💡 Premium için adminle iletişime geç!",
            parse_mode='Markdown',
            reply_markup=create_main_keyboard(user_id)
        )
        return
    
    bot.send_message(
        user_id,
        f"📤 **DOSYA YÜKLE**\n\n✨ Desteklenenler: `.py`, `.js`, `.zip`\n📦 Max boyut: 20MB\n🎯 Limit: {current_files}/20\n\n👇 Dosyanı gönder:",
        parse_mode='Markdown'
    )

def handle_my_files(message):
    """📁 Dosyalarım"""
    user_id = message.from_user.id
    user_files_list = user_files.get(user_id, [])
    
    if not user_files_list:
        bot.send_message(
            user_id,
            "📭 **HENÜZ DOSYA YOK**\n\n✨ İlk botunu yüklemek için 'YÜKLE' butonuna bas! 🚀",
            reply_markup=create_main_keyboard(user_id)
        )
        return
    
    files_text = f"📁 **DOSYALARIN** ({len(user_files_list)})\n\n"
    
    emoji_map = {
        ('approved', True): ['🚀', '⚡', '💫'],
        ('approved', False): ['⏸️', '🛑', '⏯️'],
        'pending': ['⏳', '🕒', '⏱️'],
        'rejected': ['❌', '🚫', '⛔']
    }
    
    for idx, (file_name, file_type, status) in enumerate(user_files_list, 1):
        if status == 'approved':
            is_running = is_bot_running(user_id, file_name)
            emoji = random.choice(emoji_map[(status, is_running)])
            status_text = "ÇALIŞIYOR" if is_running else "DURDU"
        elif status == 'pending':
            emoji = random.choice(emoji_map[status])
            status_text = "BEKLİYOR"
        else:
            emoji = random.choice(emoji_map[status])
            status_text = "REDDEDİLDİ"
        
        files_text += f"{emoji} `{file_name}`\n   📝 {file_type.upper()} | 📊 {status_text}\n\n"
    
    bot.send_message(
        user_id,
        files_text,
        parse_mode='Markdown',
        reply_markup=create_files_keyboard(user_id)
    )
#emirhan gel beni sik askimbah hh iahh iahhh
def handle_file_control(message, user_id, file_name, file_type, status):
    # Seçilen dosyayı kaydet
    user_states[user_id] = file_name
    """🎮 Dosya kontrol menüsü"""
    if status == 'pending':
        text = f"⏳ **BEKLİYOR**\n\n📄 `{file_name}`\n🎯 {file_type.upper()}\n\n✨ Admin onayı bekleniyor..."
    elif status == 'rejected':
        text = f"❌ **REDDEDİLDİ**\n\n📄 `{file_name}`\n\n💔 Bu dosya admin tarafından reddedildi."
    elif status == 'approved':
        is_running = is_bot_running(user_id, file_name)
        if is_running:
            text = f"🚀 **ÇALIŞIYOR**\n\n📄 `{file_name}`\n🎯 {file_type.upper()}\n\n✨ Bot aktif çalışıyor!"
        else:
            text = f"⏸️ **DURDU**\n\n📄 `{file_name}`\n🎯 {file_type.upper()}\n\n💡 Bot şu anda durdurulmuş."
    else:
        text = f"❓ **BİLİNMİYOR**\n\n📄 `{file_name}`"
    
    is_running = is_bot_running(user_id, file_name) if status == 'approved' else False
    bot.send_message(
        user_id,
        text,
        parse_mode='Markdown',
        reply_markup=create_file_control_keyboard(file_name, status, is_running)
    )

def handle_file_action(message, action):
    user_id = message.from_user.id

    user_files_list = user_files.get(user_id, [])
    if not user_files_list:
        bot.send_message(user_id, "❌ Hiç dosya bulunamadı!")
        return

    selected_file = user_states.get(user_id)
    if not selected_file:
        bot.send_message(user_id, "❌ Önce bir dosya seç!")
        return

    target_file = None
    for file_name, file_type, status in user_files_list:
        if file_name == selected_file and status == 'approved':
            target_file = (file_name, file_type)
            break

    if not target_file:
        bot.send_message(user_id, "❌ Onaylanmış dosya bulunamadı!")
        return

    file_name, file_type = target_file

    # 📂 DOSYA YOLUNU BURADA TANIMLIYORUZ (HATAYI ÇÖZEN KISIM)
    user_folder = get_user_folder(user_id)
    file_path = os.path.join(user_folder, file_name)

    # 🚀 BAŞLAT
    if action == '🚀 BAŞLAT':
        if os.path.exists(file_path):
            run_bot_with_log(user_id, file_name, file_path, file_type)
            bot.send_message(user_id, f"🚀 `{file_name}` başlatıldı!")
        else:
            bot.send_message(user_id, "❌ Dosya sistemde bulunamadı!")

    # ⏸️ DURDUR
    elif action == '⏸️ DURDUR':
        script_key = f"{user_id}_{file_name}"
        if script_key in bot_scripts:
            try:
                bot_scripts[script_key]['process'].terminate()
                bot.send_message(user_id, f"⏸️ `{file_name}` durduruldu.")
            except:
                bot.send_message(user_id, "❌ Durdurulurken bir hata oluştu!")
        else:
            bot.send_message(user_id, "❌ Bu bot zaten çalışmıyor!")

    # ... (Diğer elif blokları: YENİDEN, SİL, LOGLAR aynı kalabilir)

    # 🔄 YENİDEN
    elif action == '🔄 YENİDEN':
        script_key = f"{user_id}_{file_name}"
        if script_key in bot_scripts:
            try:
                bot_scripts[script_key]['process'].terminate()
                time.sleep(1)
            except:
                pass

        if os.path.exists(file_path):
            run_bot_with_log(user_id, file_name, file_path, file_type)
            bot.send_message(user_id, f"🔄 Yeniden başlatıldı: `{file_name}`")
        else:
            bot.send_message(user_id, "❌ Dosya yok!")

    # 🗑️ SİL
    elif action == '🗑️ SİL':
        try:
            if os.path.exists(file_path):
                os.remove(file_path)

            if user_id in user_files:
                user_files[user_id] = [
                    (fn, ft, st) for fn, ft, st in user_files[user_id]
                    if fn != file_name
                ]

            bot.send_message(user_id, f"🗑️ Silindi: `{file_name}`")
        except Exception as e:
            bot.send_message(user_id, f"❌ Hata: {str(e)[:50]}")

    # 📋 LOGLAR
    elif action == '📋 LOGLAR':
        script_key = f"{user_id}_{file_name}"
        if script_key in bot_scripts:
            bot.send_message(user_id, f"📋 `{file_name}` çalışıyor")
        else:
            bot.send_message(user_id, f"📋 `{file_name}` çalışmıyor")
    

def handle_speed(message):
    """⚡ Hız testi"""
    user_id = message.from_user.id
    
    start_time = time.time()
    
    # Kısa bir işlem yap
    time.sleep(0.1)
    
    response_time = round((time.time() - start_time) * 1000, 2)
    
    if user_id == OWNER_ID:
        user_level = "👑 SAHİP"
        level_emoji = "👑"
    elif user_id in admin_ids:
        user_level = "🔧 ADMIN"
        level_emoji = "🔧"
    else:
        user_level = "⭐ KULLANICI"
        level_emoji = "⭐"
    
    running_bots = sum(1 for s in bot_scripts.values() if s['process'].poll() is None)
    
    # Hız durumuna göre emoji
    if response_time < 100:
        speed_emoji = "⚡"
        speed_status = "SÜPER HIZLI"
    elif response_time < 300:
        speed_emoji = "🚀"
        speed_status = "HIZLI"
    elif response_time < 500:
        speed_emoji = "💨"
        speed_status = "NORMAL"
    else:
        speed_emoji = "🐢"
        speed_status = "YAVAŞ"
    
    speed_msg = (
        f"{speed_emoji} **BOT HIZI** {speed_emoji}\n\n"
        f"⏱️ Yanıt Süresi: `{response_time} ms`\n"
        f"📊 Durum: {speed_status}\n\n"
        f"{level_emoji} Seviyen: {user_level}\n"
        f"👥 Aktif Kullanıcı: `{len(active_users)}`\n"
        f"🤖 Çalışan Bot: `{running_bots}`\n"
        f"📂 Toplam Dosya: `{sum(len(files) for files in user_files.values())}`"
    )
    
    bot.send_message(user_id, speed_msg, parse_mode='Markdown')

def handle_stats(message):
    """📊 İstatistikler"""
    user_id = message.from_user.id
    
    total_users = len(active_users)
    total_files = sum(len(files) for files in user_files.values())
    running_bots = sum(1 for s in bot_scripts.values() if s['process'].poll() is None)
    pending_count = len(pending_approvals)
    
    # Veritabanından ek bilgiler
    try:
        conn = sqlite3.connect(DATABASE_PATH, check_same_thread=False)
        c = conn.cursor()
        
        # Toplam yüklenen dosya sayısı
        c.execute('SELECT COUNT(*) FROM user_files')
        total_uploads = c.fetchone()[0]
        
        # Bugün yüklenen dosyalar
        c.execute('SELECT COUNT(*) FROM user_files WHERE DATE(upload_time) = DATE("now")')
        today_uploads = c.fetchone()[0]
        
        conn.close()
    except:
        total_uploads = total_files
        today_uploads = 0
    
    stats_msg = (
        f"📊 **BOT İSTATİSTİKLERİ** 📊\n\n"
        f"👥 Toplam Kullanıcı: `{total_users}`\n"
        f"📂 Aktif Dosya: `{total_files}`\n"
        f"🚀 Çalışan Bot: `{running_bots}`\n"
        f"⏳ Bekleyen Onay: `{pending_count}`\n\n"
        f"📈 **GÜNLÜK İSTATİSTİK**\n"
        f"📤 Bugünkü Yüklemeler: `{today_uploads}`\n"
        f"📦 Toplam Yükleme: `{total_uploads}`"
    )
    
    bot.send_message(user_id, stats_msg, parse_mode='Markdown')

def handle_pending_approvals(message):
    """👑 Bekleyen onaylar (admin)"""
    user_id = message.from_user.id
    
    if user_id not in admin_ids:
        bot.send_message(user_id, "❌ Bu işlem için yetkin yok!")
        return
    
    if not pending_approvals:
        bot.send_message(user_id, "✅ Şu anda bekleyen onay yok!")
        return
    
    text = f"⏳ **BEKLEYEN ONAYLAR** ({len(pending_approvals)})\n\n"
    
    for file_id, info in list(pending_approvals.items())[:5]:  # İlk 5'i göster
        text += (
            f"👤 `{info['user_name']}`\n"
            f"📄 `{info['file_name']}`\n"
            f"🎯 {info['file_type'].upper()}\n"
            f"⏰ {info['upload_time'].strftime('%H:%M')}\n"
            f"────────────\n"
        )
    
    if len(pending_approvals) > 5:
        text += f"\n... ve {len(pending_approvals) - 5} daha bekliyor."
    
    bot.send_message(user_id, text, parse_mode='Markdown')

# ================================
# 🌟 DOSYA YÜKLEME HANDLER 🌟
# ================================
@bot.message_handler(content_types=['document'])
def handle_document(message):
    user_id = message.from_user.id
    
    # Doğrulama kontrolü
    if user_id not in verified_users and user_id not in admin_ids:
        bot.send_message(
            user_id,
            "❌ Önce kanal doğrulaması yapmalısın!\n\n/start yazarak başlayabilirsin.",
            reply_markup=types.ReplyKeyboardRemove()
        )
        return
    
    doc = message.document
    file_name = doc.file_name
    
    if not file_name:
        bot.reply_to(message, "❌ Dosya adı yok!")
        return
    
    file_ext = os.path.splitext(file_name)[1].lower()
    if file_ext not in ['.py', '.js', '.zip']:
        bot.reply_to(message, "❌ Sadece `.py`, `.js`, `.zip` dosyaları kabul edilir!")
        return
    
    # Limit kontrolü
    current_files = get_user_file_count(user_id)
    if current_files >= 20:
        bot.reply_to(message, f"❌ Dosya limiti doldu! ({current_files}/20)")
        return
    
    # Dosya boyutu kontrolü (20MB limit)
    if doc.file_size > 20 * 1024 * 1024:
        bot.reply_to(message, "❌ Dosya çok büyük! Max 20MB")
        return
    
    try:
        bot.reply_to(message, f"📥 **YÜKLENİYOR**\n\n`{file_name}`\n\n⏳ Lütfen bekle...")
        
        # Dosyayı indir
        file_info = bot.get_file(doc.file_id)
        downloaded_file = bot.download_file(file_info.file_path)
        
        user_folder = get_user_folder(user_id)
        
        if file_ext == '.zip':
            # ZIP dosyasını işle
            handle_zip_file(downloaded_file, file_name, message, user_id, user_folder)
        else:
            # Normal dosyayı kaydet
            file_path = os.path.join(user_folder, file_name)
            with open(file_path, 'wb') as f:
                f.write(downloaded_file)
            
            file_type = 'py' if file_ext == '.py' else 'js'
            
            # Veritabanına kaydet
            if save_user_file(user_id, file_name, file_type, 'pending'):
                # Admin onayı için bekleyenlere ekle
                file_id = f"{user_id}_{file_name}_{int(time.time())}"
                pending_approvals[file_id] = {
                    'user_id': user_id,
                    'user_name': message.from_user.first_name,
                    'file_name': file_name,
                    'file_type': file_type,
                    'file_path': file_path,
                    'upload_time': datetime.now()
                }
                
                # Admin'e bildir
                admin_msg = (
                    f"📤 **YENİ DOSYA**\n\n"
                    f"👤 {message.from_user.first_name}\n"
                    f"🆔 {user_id}\n"
                    f"📄 {file_name}\n"
                    f"🎯 {file_type.upper()}"
                )
                
                try:
                    with open(file_path, 'rb') as f:
                        bot.send_document(
                            ADMIN_ID,
                            f,
                            caption=admin_msg,
                            reply_markup=create_approval_inline_keyboard(file_id)
                        )
                except:
                    bot.send_message(
                        ADMIN_ID,
                        admin_msg,
                        reply_markup=create_approval_inline_keyboard(file_id)
                    )
                
                bot.reply_to(
                    message,
                    f"✅ **YÜKLENDİ!**\n\n`{file_name}`\n\n⏳ Admin onayı bekleniyor..."
                )
            else:
                bot.reply_to(message, "❌ Kayıt sırasında hata oluştu!")
        
    except Exception as e:
        logger.error(f"❌ Dosya yükleme hatası: {e}", exc_info=True)
        bot.reply_to(message, f"❌ Hata: {str(e)[:100]}")

def handle_zip_file(downloaded_file_content, file_name_zip, message, user_id, user_folder):
    """🗜️ ZIP dosyasını işle"""
    temp_dir = None
    
    try:
        temp_dir = tempfile.mkdtemp(prefix=f"zip_{user_id}_")
        zip_path = os.path.join(temp_dir, file_name_zip)
        
        # ZIP'i kaydet
        with open(zip_path, 'wb') as f:
            f.write(downloaded_file_content)
        
        # ZIP'i aç
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            zip_ref.extractall(temp_dir)
        
        # Script dosyalarını bul
        extracted = os.listdir(temp_dir)
        py_files = [f for f in extracted if f.lower().endswith('.py')]
        js_files = [f for f in extracted if f.lower().endswith('.js')]
        
        # Ana script'i belirle
        main_script = None
        file_type = None
        
        # Öncelik sırası
        for name in ['main.py', 'bot.py', 'app.py', 'index.py']:
            if name in py_files:
                main_script = name
                file_type = 'py'
                break
        
        if not main_script:
            for name in ['index.js', 'main.js', 'bot.js', 'app.js']:
                if name in js_files:
                    main_script = name
                    file_type = 'js'
                    break
        
        if not main_script:
            if py_files:
                main_script = py_files[0]
                file_type = 'py'
            elif js_files:
                main_script = js_files[0]
                file_type = 'js'
        
        if not main_script:
            bot.reply_to(message, "❌ ZIP'te script dosyası bulunamadı!")
            return
        
        # Dosyaları taşı
        for item in os.listdir(temp_dir):
            src = os.path.join(temp_dir, item)
            dst = os.path.join(user_folder, item)
            
            if os.path.exists(dst):
                try:
                    if os.path.isfile(dst):
                        os.remove(dst)
                    else:
                        shutil.rmtree(dst)
                except:
                    pass
            
            try:
                if os.path.isfile(src):
                    shutil.copy2(src, dst)
                else:
                    shutil.copytree(src, dst)
            except Exception as e:
                logger.error(f"❌ Dosya kopyalama hatası {item}: {e}")
        
        # Veritabanına kaydet
        if save_user_file(user_id, main_script, file_type, 'pending'):
            file_path = os.path.join(user_folder, main_script)
            file_id = f"{user_id}_{main_script}_{int(time.time())}"
            
            pending_approvals[file_id] = {
                'user_id': user_id,
                'user_name': message.from_user.first_name,
                'file_name': main_script,
                'file_type': file_type,
                'file_path': file_path,
                'upload_time': datetime.now()
            }
            
            # Admin'e bildir
            admin_msg = (
                f"📦 **YENİ ZIP**\n\n"
                f"👤 {message.from_user.first_name}\n"
                f"🆔 {user_id}\n"
                f"📄 {main_script}\n"
                f"🎯 {file_type.upper()}\n"
                f"🗜️ {file_name_zip}"
            )
            
            try:
                with open(file_path, 'rb') as f:
                    bot.send_document(
                        ADMIN_ID,
                        f,
                        caption=admin_msg,
                        reply_markup=create_approval_inline_keyboard(file_id)
                    )
            except:
                bot.send_message(
                    ADMIN_ID,
                    admin_msg,
                    reply_markup=create_approval_inline_keyboard(file_id)
                )
            
            bot.reply_to(
                message,
                f"✅ **ZIP AÇILDI!**\n\n📄 `{main_script}`\n🎯 {file_type.upper()}\n\n⏳ Admin onayı bekleniyor..."
            )
        else:
            bot.reply_to(message, "❌ Kayıt sırasında hata!")
        
    except zipfile.BadZipFile:
        bot.reply_to(message, "❌ Geçersiz ZIP dosyası!")
    except Exception as e:
        logger.error(f"❌ ZIP işleme hatası: {e}", exc_info=True)
        bot.reply_to(message, f"❌ ZIP hatası: {str(e)[:100]}")
    finally:
        if temp_dir and os.path.exists(temp_dir):
            try:
                shutil.rmtree(temp_dir)
            except:
                pass

# ================================
# 🌟 TEMİZLİK FONKSİYONU 🌟
# ================================
def cleanup():
    """🧹 Çıkışta temizlik"""
    logger.warning("🔴 Bot kapatılıyor...")
    
    # Tüm botları durdur
    for script_key, script_info in list(bot_scripts.items()):
        try:
            proc = script_info.get('process')
            if proc and proc.poll() is None:
                proc.terminate()
                logger.info(f"⏹️ Bot durduruldu: {script_key}")
        except:
            pass
    
    bot_scripts.clear()
    logger.info("✅ Temizlik tamamlandı!")

atexit.register(cleanup)

# ================================
# 🌟 ANA ÇALIŞTIRMA 🌟
# ================================
if __name__ == '__main__':
    # GÜZEL BAŞLANGIÇ EKRANI
    startup_messages = [
        "✨ BOT BAŞLATILIYOR... ✨",
        "🚀 SİSTEM HAZIRLANIYOR... 🚀",
        "🌟 BOT AKTİF EDİLİYOR... 🌟"
    ]
    
    logger.info("="*50)
    logger.info(f"🎯 {random.choice(startup_messages)}")
    logger.info("="*50)
    logger.info(f"🔑 Token: {TOKEN[:10]}...")
    logger.info(f"👑 Owner: {OWNER_ID}")
    logger.info(f"🔧 Admin: {ADMIN_ID}")
    logger.info(f"📊 Veritabanı: {DATABASE_PATH}")
    logger.info(f"📂 Klasör: {UPLOAD_BOTS_DIR}")
    logger.info("="*50)
    
    
    try:
        bot_info = bot.get_me()
        logger.info(f"🤖 Bot: @{bot_info.username}")
        logger.info(f"🆔 Bot ID: {bot_info.id}")
        logger.info("✅ Bot bağlantısı başarılı!")
    except Exception as e:
        logger.error(f"❌ Bot bağlantı hatası: {e}")
        exit(1)
    
    # Polling başlat
    logger.info("🔄 Polling başlatılıyor...")
    
    while True:
        try:
            bot.polling(none_stop=True, interval=1, timeout=30)
        except Exception as e:
            logger.critical(f"❌ KRİTİK HATA: {e}")
            logger.info("🔄 10 saniye sonra yeniden başlatılıyor...")
            time.sleep(10)

            from flask import Flask
import threading

app = Flask('')

@app.route('/')
def home():
    return "Bot Calisiyor!"

def run_web():
    app.run(host='0.0.0.0', port=8080)

# Botu başlatmadan önce web sunucusunu yan kolda çalıştır
if __name__ == "__main__":
    threading.Thread(target=run_web).start()
    bot.infinity_polling()
