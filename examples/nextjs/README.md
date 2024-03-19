import MetaTrader5 as mt5
import time
import logging

# Configure logging
logging.basicConfig(filename='copy_trading.log', level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Sample user credentials (replace with your actual authentication system)
registered_users = {
    "username1": {"password": "password1", "authorized": True, "client_account_number": "123456", "risk_factor": 0.5},
    "username2": {"password": "password2", "authorized": False, "client_account_number": "789012", "risk_factor": 1.0},
}

def login(username, password):
    if username in registered_users and registered_users[username]["password"] == password:
        return registered_users[username]["authorized"]
    return False

def copy_trade_to_client(trade, client_account_number):
    # Get trade details
    symbol = trade.symbol
    action = trade.action
    volume = trade.volume
    price = trade.price
    stop_loss = trade.tp
    take_profit = trade.sl
    
    # Apply risk factor
    risk_factor = registered_users[username]["risk_factor"]
    volume *= risk_factor
    
    # Place trade on client's account
    result = mt5.order_send(symbol=symbol, action=action, volume=volume, price=price, sl=stop_loss, tp=take_profit, magic=0, comment="Copied trade")
    if result.retcode != mt5.TRADE_RETCODE_DONE:
        logging.error(f"Failed to copy trade to client {client_account_number}. Error: {result.comment}")
        return False
    else:
        logging.info(f"Trade copied successfully to client {client_account_number}")
        return True

def monitor_trades():
    # Monitor trades made by your account and copy them to clients' accounts
    while True:
        # Replace this with your logic to monitor trades (e.g., check for new trades)
        trades = mt5.history_deals_get()
        
        # Copy trades to connected clients' accounts
        for trade in trades:
            for username, user_info in registered_users.items():
                if user_info["authorized"]:
                    client_account_number = user_info["client_account_number"]
                    if copy_trade_to_client(trade, client_account_number):
                        time.sleep(1)  # Sleep to avoid overloading the server
        
        time.sleep(5)  # Adjust the sleep interval as needed

# Connect to MetaTrader 5
if not mt5.initialize():
    logging.error("Failed to connect to MetaTrader 5")
    mt5.shutdown()
else:
    logging.info("Connected to MetaTrader 5")

    # Login
    username = input("Enter your username: ")
    password = input("Enter your password: ")

    if login(username, password):
        logging.info("Login successful")
        
        # Check authorization before performing actions
        if registered_users[username]["authorized"]:
            # Now you can use MetaTrader 5 functions to interact with the platform
            # For example, you can get account information:
            account_info = mt5.account_info()
            logging.info(f"Account info: {account_info}")
            
            # Start monitoring trades
            monitor_trades()
        else:
            logging.warning("You are not authorized to perform this action.")
    else:
        logging.error("Login failed. Please check your username and password.")

    # Remember to shutdown the MetaTrader 5 connection when done
    mt5.shutdown()
