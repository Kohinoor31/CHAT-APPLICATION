# CHAT-APPLICATION
import tkinter as tk
import socket
import threading

class ChatApp:
    def __init__(self, master):
        self.master = master
        self.master.title("Chat Application")
        self.master.geometry("400x500")

        self.text_area = tk.Text(self.master, height=15, width=50)
        self.text_area.pack(pady=20)

        self.entry_box = tk.Entry(self.master, width=50)
        self.entry_box.pack()

        self.send_button = tk.Button(self.master, text="Send", command=self.send_message)
        self.send_button.pack(pady=5)

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.bind(('localhost', 12345))
        self.server.listen()

        self.clients = []
        self.nicknames = []

        self.thread = threading.Thread(target=self.handle_clients)
        self.thread.start()

    def receive_message(self, client_socket):
        while True:
            try:
                message = client_socket.recv(1024).decode('utf-8')
                if message == 'NICK':
                    client_socket.send(self.nickname.encode('utf-8'))
                else:
                    self.text_area.insert(tk.END, f'{client_socket.getpeername()[0]} > {message}\n')
                    self.text_area.see(tk.END)
            except:
                print("An error occurred while receiving messages.")
                client_socket.close()
                self.clients.remove(client_socket)
                break

    def send_message(self):
        message = f'{self.nickname}: {self.entry_box.get()}'
        for client in self.clients:
            client.send(message.encode('utf-8'))

    def handle_clients(self):
        while True:
            client, address = self.server.accept()
            print(f"Connected with {str(address)}")

            self.clients.append(client)

            thread = threading.Thread(target=self.receive_message, args=(client,))
            thread.start()

if __name__ == "__main__":
    root = tk.Tk()

    app = ChatApp(root)

    while True:
        nickname = input("Enter your nickname: ")
        if nickname:
            app.nickname = nickname
            root.mainloop()
            break

    root.destroy()
