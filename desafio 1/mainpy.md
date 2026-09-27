import re

def identificar_bandeira(numero_cartao):
    # Remove espaços em branco ou hífens que o usuário possa digitar
    numero_limpo = re.sub(r'\D', '', numero_cartao)
    
    # Expressões regulares (Regex) para cada bandeira com base nos prefixos comuns
    padroes = {
        "Visa": r"^4[0-9]{12}(?:[0-9]{3})?$",
        "MasterCard": r"^(5[1-5][0-9]{2}|2[2-7][0-9]{2})[0-9]{12}$",
        "American Express": r"^3[47][0-9]{13}$",
        "Discover": r"^6011[0-9]{12}$|^65[0-9]{14}$",
        "Elo": r"^(401178|401179|431274|438935|451416|457393|457631|404115|506699|5067[0-6][0-9]|5090[0-9]{3}|6500[3-5][0-9]|6504[0-3][0-9]|65048[5-9]|6505[0-3][0-9]|6505[4-8][0-9]|6505[9][0-9]|6507[0-2][0-9]|6509[0-x][0-9]|6516[5-7][0-9]|6550[0-1][0-9]|6550[2-3][0-9])[0-9]{10}$"
    }
    
    for bandeira, regex in padroes.items():
        if re.match(regex, numero_limpo):
            return bandeira
            
    return "Bandeira desconhecida"

if __name__ == "__main__":
    print("=== Identificador de Bandeira de Cartão de Crédito ===")
    cartao = input("Digite o número do cartão de crédito: ")
    resultado = identificar_bandeira(cartao)
    print(f"A bandeira do cartão é: **{resultado}**")
