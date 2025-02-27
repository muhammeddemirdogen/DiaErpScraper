import requests
from bs4 import BeautifulSoup
import os
import schedule
import time
import smtplib
from email.mime.text import MIMEText

# Sabitler
DOSYA = "onceki_duyurular.txt"
BASE_URL = "https://www.diaakademi.com/gelistirme-duyurulari/?_page="
GONDEREN = "basari.printer@gmail.com"
ALICI = "muhammed@basari.com.tr"
SIFRE = "gtrpbahxdvixyqdq"

def email_gonder(yeni_duyurular):
    # HTML formatında e-posta içeriği
    html_content = """
    <h3>Yeni Duyurular:</h3>
    <table border="1" cellspacing="0" cellpadding="5">
        <tr>
            <th>Konu</th>
            <th>Tarih</th>
            <th>Kategoriler</th>
            <th>Versiyon</th>
        </tr>
    """
    
    for baslik, tarih, href, kategoriler, etiketler in yeni_duyurular:
        kategoriler = [kat for kat in kategoriler if kat != "Geliştirme Duyuruları"]
        kategori_str = ", ".join(kategoriler) if kategoriler else "-"
        etiket_str = ", ".join(etiketler) if etiketler else "-"
        html_content += f"""
        <tr>
            <td><a href="{href}">{baslik}</a></td>
            <td>{tarih}</td>
            <td>{kategori_str}</td>
            <td>{etiket_str}</td>
        </tr>
        """
    
    html_content += "</table>"
    
    mesaj = MIMEText(html_content, "html")
    mesaj["Subject"] = "DİA ERP Yeni Duyurular"
    mesaj["From"] = GONDEREN
    mesaj["To"] = ALICI

    try:
        with smtplib.SMTP("smtp.gmail.com", 587) as server:
            server.starttls()
            server.login(GONDEREN, SIFRE)
            server.sendmail(GONDEREN, ALICI, mesaj.as_string())
            print("E-posta gönderildi!")
    except Exception as e:
        print(f"E-posta gönderimi başarısız: {e}")

def kontrol_et():
    # Önceki duyuruları oku
    if os.path.exists(DOSYA):
        with open(DOSYA, "r", encoding="utf-8") as f:
            onceki_duyurular = set(f.read().splitlines())
    else:
        onceki_duyurular = set()

    yeni_duyurular = []
    toplam_sayfa = 5

    for sayfa in range(1, toplam_sayfa + 1):
        url = f"{BASE_URL}{sayfa}"
        try:
            response = requests.get(url)
            response.raise_for_status()
        except requests.RequestException as e:
            print(f"Sayfa {sayfa} indirilemedi: {e}")
            continue

        soup = BeautifulSoup(response.content, "html.parser")
        duyuru_divleri = soup.find_all("div", class_="pt-cv-title")
        duyuru_linkleri = [div.find("a", class_="_self") for div in duyuru_divleri if div.find("a", class_="_self")]

        for link in duyuru_linkleri:
            if link:
                baslik = link.text.strip()
                href = link.get("href")
                
                parent = link.find_parent()
                tarih_span = parent.find("span", class_="entry-date") if parent else None
                if not tarih_span:
                    for span in soup.find_all("span", class_="entry-date"):
                        if span.find_previous("div", class_="pt-cv-title") == parent:
                            tarih_span = span
                            break
                tarih = tarih_span.find("time").text.strip() if tarih_span else "Tarih bulunamadı"
                
                # Kategori ve etiket bilgilerini çek
                terms_span = parent.find_next("span", class_="terms") if parent else None
                if not terms_span:
                    terms_span = soup.find("span", class_="terms")
                kategori_ve_etiketler = terms_span.find_all("a") if terms_span else []
                kategoriler = [a.text.strip() for a in kategori_ve_etiketler if "kategori" in a["href"]]
                etiketler = [a.text.strip() for a in kategori_ve_etiketler if "tag" in a["href"]]
                
                duyuru_id = f"{baslik} - {tarih}"
                if baslik and duyuru_id not in onceki_duyurular:
                    yeni_duyurular.append((baslik, tarih, href, kategoriler, etiketler))
                    onceki_duyurular.add(duyuru_id)

    if yeni_duyurular:
        print("Yeni Duyurular:")
        for baslik, tarih, href, kategoriler, etiketler in yeni_duyurular:
            kategoriler = [kat for kat in kategoriler if kat != "Geliştirme Duyuruları"]
            kategori_str = ", ".join(kategoriler) if kategoriler else "-"
            etiket_str = ", ".join(etiketler) if etiketler else "-"
            print(f"{baslik} ({tarih}) - Kategoriler: {kategori_str}, Versiyon: {etiket_str} - Link: {href}")
        email_gonder(yeni_duyurular)
    else:
        print("Yeni duyuru yok.")

    with open(DOSYA, "w", encoding="utf-8") as f:
        for duyuru in onceki_duyurular:
            f.write(duyuru + "\n")

kontrol_et()

schedule.every().hour.do(kontrol_et)

while True:
    schedule.run_pending()
    time.sleep(1)
