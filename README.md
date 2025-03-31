import os
import requests
import zipfile
from bs4 import BeautifulSoup

def baixar_pdf(url, nome_arquivo):
    resposta = requests.get(url, stream=True)
    if resposta.status_code == 200:
        with open(nome_arquivo, 'wb') as f:
            for chunk in resposta.iter_content(1024):
                f.write(chunk)
        print(f"Download concluído: {nome_arquivo}")
    else:
        print(f"Erro ao baixar {url}")

def compactar_arquivos(arquivos, nome_zip):
    with zipfile.ZipFile(nome_zip, 'w') as zipf:
        for arquivo in arquivos:
            zipf.write(arquivo)
            os.remove(arquivo)  # Remove os arquivos após a compactação
    print(f"Arquivos compactados em {nome_zip}")

def main():
    url_base = "https://www.gov.br/ans/pt-br/acesso-a-informacao/participacao-dasociedade/atualizacao-do-rol-de-procedimentos"
    resposta = requests.get(url_base)
    if resposta.status_code != 200:
        print("Erro ao acessar o site")
        return
    
    soup = BeautifulSoup(resposta.text, 'html.parser')
    links = soup.find_all('a', href=True)
    
    anexos = {}
    for link in links:
        href = link['href']
        if "Anexo_I" in href:
            anexos["Anexo_I.pdf"] = href
        elif "Anexo_II" in href:
            anexos["Anexo_II.pdf"] = href
    
    if not anexos:
        print("Nenhum anexo encontrado!")
        return
    
    arquivos_baixados = []
    for nome_arquivo, url_pdf in anexos.items():
        baixar_pdf(url_pdf, nome_arquivo)
        arquivos_baixados.append(nome_arquivo)
    
    compactar_arquivos(arquivos_baixados, "anexos.zip")
    
if __name__ == "__main__":
    main()

