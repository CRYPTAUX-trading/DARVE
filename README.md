import requests
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
import streamlit as st
from googletrans import Translator
import os
import logging
from functools import lru_cache

# Configuration du logging pour enregistrer les erreurs
logging.basicConfig(filename='app.log', format='%(asctime)s - %(message)s', level=logging.ERROR)

# Clé d'activation (utilisation d'une variable d'environnement sécurisée recommandée)
ACTIVATION_KEY = os.getenv('ACTIVATION_KEY')

def check_activation_key():
    """
    Vérifie si la clé d'activation entrée par l'utilisateur est valide.
    Affiche une erreur si la clé est incorrecte.
    """
    key = st.text_input("Enter activation key:", type="password")
    if key == ACTIVATION_KEY:
        st.success("Activation successful!")
        return True
    else:
        st.error("Invalid activation key")
        return False

# Cache pour les données des cryptomonnaies
@lru_cache(maxsize=10)
def fetch_cached_crypto_data(crypto):
    return fetch_crypto_data(crypto)

def fetch_crypto_data(crypto):
    """
    Récupère les données des cryptomonnaies à partir d'une source externe ou simule des données pour certaines cryptomonnaies.
    Affiche une erreur en cas d'échec de récupération des données.
    """
    try:
        # Vérification de la validité de la cryptomonnaie
        valid_cryptos = [
            "Bitcoin", "Ethereum", "Binance Coin", "Cardano", "Solana", 
            "XRP", "Polkadot", "Dogecoin", "Chainlink", "Litecoin",
            "Bitcoin Cash", "Stellar", "VeChain", "EOS", "Tron",
            "Monero", "Aave", "Uniswap", "Cosmos", "PancakeSwap",
            "Algorand", "Theta Network", "Filecoin", "Tezos", "Avalanche",
            "Synthetix", "Compound", "Maker", "SushiSwap", "Terra",
            "Kusama", "FTX Token", "Huobi Token", "Ethereum Classic", "Polygon",
            "The Graph", "Shiba Inu", "Decentraland", "Quant", "Dash",
            "Zcash", "Waves", "Ren", "Basic Attention Token", "0x",
            "Golem", "VeThor Token", "NEM", "Ontology", "Siacoin"
        ]
        if crypto.capitalize() not in valid_cryptos:
            raise ValueError(f"Invalid cryptocurrency: {crypto}")
        
        if crypto in ["bitcoin", "ethereum", "litecoin", "ripple", "bitcoin-cash", "ethereum-classic"]:
            # Simulation de données pour certaines cryptomonnaies
            data = {
                "name": [crypto.capitalize()] * 100,
                "price": np.random.rand(100) * 1000
            }
            df = pd.DataFrame(data)
            df['name'] = f"{crypto.capitalize()}"
            return df.sort_values(by='price')
        else:
            # Récupération de données depuis une API externe (exemple hypothétique)
            url = f"https://courscryptomonnaies.com/{crypto}"
            response = requests.get(url)
            response.raise_for_status()  # Vérifier si la requête a réussi
            
            if response.status_code == 200:
                data = response.json()
                df = pd.DataFrame(data)
                return df.sort_values(by='price')
            else:
                st.error(f"Error fetching data for {crypto}: {response.status_code} - {response.text}")
                logging.error(f"Error fetching data for {crypto}: {response.status_code} - {response.text}")
                return pd.DataFrame()
    except requests.exceptions.RequestException as e:
        st.error(f"Error fetching data for {crypto}: {e}")
        logging.error(f"Error fetching data for {crypto}: {e}")
        return pd.DataFrame()
    except ValueError as ve:
        st.error(f"Invalid cryptocurrency selected: {crypto}")
        logging.error(f"Invalid cryptocurrency selected: {crypto}")
        return pd.DataFrame()
    except Exception as ex:
        st.error(f"Unexpected error fetching data for {crypto}: {ex}")
        logging.error(f"Unexpected error fetching data for {crypto}: {ex}")
        return pd.DataFrame()

def predict_future_prices(crypto, days, historical_data=None):
    """
    Prédit les prix futurs des cryptomonnaies en utilisant un modèle de régression linéaire.
    Affiche une erreur en cas d'échec de prédiction.
    """
    try:
        if historical_data is None:
            # Simulation de données historiques pour l'exemple
            dates = pd.date_range(start="2023-01-01", periods=100)
            prices = np.random.rand(100) * 1000
            historical_data = pd.DataFrame({'Date': dates, 'Price': prices})
        
        historical_data['Day'] = historical_data.index
        X = historical_data[['Day']]
        y = historical_data['Price']
        
        model = LinearRegression()
        model.fit(X, y)
        
        future_day = len(historical_data) + days
        future_price = model.predict(np.array([[future_day]]))
        return future_price[0]
    except Exception as e:
        st.error(f"Error predicting future price for {crypto}: {e}")
        logging.error(f"Error predicting future price for {crypto}: {e}")
        return None

def translate_text(text, lang):
    """
    Traduit le texte donné dans la langue spécifiée.
    Affiche une erreur en cas d'échec de traduction.
    """
    try:
        if not hasattr(translate_text, 'translator'):
            translate_text.translator = Translator()
        translation = translate_text.translator.translate(text, dest=lang)
        return translation.text
    except Exception as e:
        st.error(f"Translation error: {e}")
        logging.error(f"Translation error: {e}")
        return text

def main():
    """
    Fonction principale pour l'interface utilisateur avec Streamlit.
    """
    if not check_activation_key():
        return
    
    st.title("Crypto Price Estimation using AI")
    
    lang = st.selectbox("Choose your language", ["en", "fr", "ar", "es"])
    
    cryptos = [
        "Bitcoin", "Ethereum", "Binance Coin", "Cardano", "Solana", 
        "XRP", "Polkadot", "Dogecoin", "Chainlink", "Litecoin",
        "Bitcoin Cash", "Stellar", "VeChain", "EOS", "Tron",
        "Monero", "Aave", "Uniswap", "Cosmos", "PancakeSwap",
        "Algorand", "Theta Network", "Filecoin", "Tezos", "Avalanche",
        "Synthetix", "Compound", "Maker", "SushiSwap", "Terra",
        "Kusama", "FTX Token", "Huobi Token", "Ethereum Classic", "Polygon",
        "The Graph", "Shiba Inu", "Decentraland", "Quant", "Dash",
        "Zcash", "Waves", "Ren", "Basic Attention Token", "0x",
        "Golem", "VeThor Token", "NEM", "Ontology", "Siacoin"
    ]
    
    st.subheader(translate_text("Search for Cryptocurrencies", lang))
    selected_crypto = st.selectbox(translate_text("Select Cryptocurrency", lang), cryptos)
    
    # Vérifier si la cryptomonnaie sélectionnée est dans la liste cryptos
    if selected_crypto not in cryptos:
        st.error("Invalid cryptocurrency selected.")
        return
    
    df = fetch_cached_crypto_data(selected_crypto.lower().replace(" ", "-"))
    
    if not df.empty:
        st.dataframe(df)
    
        days_option = st.selectbox(translate_text("Select future date", lang), ["1 day", "1 week", "1 month", "1 year"])
        days_map = {"1 day": 1, "1 week": 7, "1 month": 30, "1 year": 365}
        days = days_map[days_option]
        
        future_price = predict_future_prices(selected_crypto.lower().replace(" ", "-"), days)
        if future_price is not None:
            st.write(translate_text(f"Predicted price in {days_option}: ${future_price:.2f}", lang))
        
        st.subheader(translate_text("Current Prices and Graphs", lang))
        st.line_chart(df.set_index('name')['price'])

if __name__ == "__main__":
    main()

