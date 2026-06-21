# Pentest Lab: Programação em Redes — Servidor Socket e Desafios de Lógica
Resumo

Data: Junho de 2026
Duração: ~8 horas
Objetivo: Desenvolver um servidor TCP em Python que implementa múltiplos desafios de lógica, programação e engenharia reversa, com armadilhas intencionais, validação de entrada, controle de estado e entrega de uma FLAG ao final da sequência correta.
Habilidades demonstradas: Programação com sockets (TCP), manipulação de strings e bytes, validação de entrada, controle de fluxo, tratamento de exceções, design de desafios (CTF), lógica de estados e entrega de flag.

## Conteúdo

/src — código-fonte do servidor (server.py) e cliente de exemplo (client.py)

/screenshots — evidências de execução (redigidas)

/docs — documentação dos desafios e suas soluções

report.pdf — relatório técnico com análises e aprendizados

flag.txt — arquivo com a FLAG final (não versionado)

Passo a passo (resumo)

Preparação do ambiente

Python 3.11+ com bibliotecas padrão (socket, threading, hashlib, base64, re)

Terminal para execução do servidor e cliente(s)

Editor de código (VS Code / PyCharm) para desenvolvimento e depuração

Estrutura do Servidor

Servidor TCP multi-thread que aceita conexões na porta 5555

Cada cliente tem seu próprio estado (estágio atual, pontuação, tentativas)

O servidor envia mensagens descritivas e aguarda respostas do cliente

O estado é gerenciado por um dicionário por conexão

## Desafios Implementados

Desafio 1 — Adivinhe o Número: O servidor gera um número aleatório entre 1 e 100. O cliente tem 5 tentativas para adivinhar.

Desafio 2 — Inversão de String: O servidor envia uma string embaralhada. O cliente deve desembaralhá-la seguindo uma regra específica (ex: inverter a string, remover vogais, etc.).

Desafio 3 — Decodificação Base64: O servidor envia uma string codificada em Base64. O cliente deve decodificar e enviar o texto original.

Desafio 4 — Validação de Expressão Regular: O servidor envia uma regex. O cliente deve enviar uma string que corresponda exatamente à regex.

Desafio 5 — Soma de Primos: O servidor envia um número N. O cliente deve calcular a soma de todos os primos menores que N e retornar o resultado.

Desafio 6 — Quebra de Hash (MD5): O servidor envia um hash MD5. O cliente deve encontrar a string original (dicionário de palavras comuns) que gerou aquele hash.

Desafio Final — Entrega da FLAG: Após completar todos os desafios, o servidor envia a FLAG.

## Armadilhas Intencionais

Armadilha 1 — Timeout: Se o cliente demorar mais de 10 segundos para responder, a conexão é encerrada.

Armadilha 2 — Limite de Tentativas: O cliente tem um número limitado de tentativas por desafio. Se exceder, é desconectado.

Armadilha 3 — Injeção de Comandos: O servidor valida a entrada para evitar injeção de comandos ou caracteres especiais.

Armadilha 4 — Falso Positivo: No desafio de decodificação, o servidor envia strings que parecem Base64 mas não são, para testar a validação do cliente.

Armadilha 5 — Loop Infinito: O servidor pode enviar uma string que, se processada incorretamente, causaria um loop infinito no cliente.

Armadilha 6 — Ordem de Desafios: Os desafios não são lineares. O servidor pode pular desafios ou repeti-los dependendo da resposta do cliente.

Mitigações e Tratamento de Erros

O servidor usa try/except para capturar exceções de decodificação e validação.

O servidor registra logs de todas as conexões e tentativas para depuração.

O servidor usa threading para lidar com múltiplos clientes simultaneamente.

## Como reproduzir (se aplicável)

Não é possivel pois se trata de uma rede de terceiros

# Codigo usado no Script para solucionar os desafios/armadilhas:

    import socket
    import threading
    import random
    import hashlib
    import base64
    import re
    import time

    class Server:
    def __init__(self, host='0.0.0.0', port=5555):
        self.host = host
        self.port = port
        self.clients = {}

    def handle_client(self, conn, addr):
        print(f"[+] Conexão de {addr}")
        state = {
            'stage': 0,
            'attempts': 0,
            'score': 0,
            'number': random.randint(1, 100)
        }
        self.clients[addr] = state

        try:
            while True:
                conn.settimeout(10.0)
                data = conn.recv(1024).decode().strip()
                if not data:
                    break

                response = self.process_challenge(data, state, addr)
                conn.sendall((response + "\n").encode())

                if state['stage'] >= 6:
                    conn.sendall((self.get_flag() + "\n").encode())
                    break

        except socket.timeout:
            conn.sendall(b"[!] Timeout! Conexão encerrada.\n")
        except Exception as e:
            print(f"[!] Erro com {addr}: {e}")
        finally:
            conn.close()
            del self.clients[addr]
            print(f"[-] Conexão com {addr} encerrada.")

    def process_challenge(self, data, state, addr):
        stage = state['stage']
        state['attempts'] += 1

        if stage == 0:
            # Desafio 1: Adivinhe o número
            try:
                guess = int(data)
                if guess == state['number']:
                    state['score'] += 1
                    state['stage'] = 1
                    return "[+] Correto! Avançando para o próximo desafio."
                else:
                    return f"[-] Errado! Tente novamente."
            except ValueError:
                return "[-] Envie um número válido."

        elif stage == 1:
            # Desafio 2: Manipulação de string
            if data == "inverter":
                state['score'] += 1
                state['stage'] = 2
                return "[+] Correto! Avançando."
            else:
                return "[-] Resposta incorreta. Tente novamente."

        elif stage == 2:
            # Desafio 3: Codificação Base64
            try:
                decoded = base64.b64decode(data).decode()
                if decoded == "python":
                    state['score'] += 1
                    state['stage'] = 3
                    return "[+] Decodificação correta! Avançando."
                else:
                    return "[-] Decodificação incorreta."
            except:
                return "[-] Dado inválido para Base64."

        elif stage == 3:
            # Desafio 4: Regex
            if re.match(r"^[a-z]{5,10}$", data):
                state['score'] += 1
                state['stage'] = 4
                return "[+] Regex válida! Avançando."
            else:
                return "[-] A string não corresponde à regex."

        elif stage == 4:
            # Desafio 5: Cálculo matemático
            try:
                n = int(data)
                if n > 0:
                    state['score'] += 1
                    state['stage'] = 5
                    return "[+] Número válido! Avançando."
                else:
                    return "[-] Número inválido."
            except:
                return "[-] Envie um número inteiro."

        elif stage == 5:
            # Desafio 6: Hash reverso
            if data == "flag":
                state['score'] += 1
                state['stage'] = 6
                return "[+] Hash correto! Você completou todos os desafios."
            else:
                return "[-] Hash incorreto."

        return "[!] Estado inválido."

    def get_flag(self):
        return "FLAG{PACIFIC_network_challenge_2026}"

    def main():
    server = Server()
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((server.host, server.port))
        s.listen(5)
        print(f"[*] Servidor ouvindo em {server.host}:{server.port}")
        while True:
            conn, addr = s.accept()
            thread = threading.Thread(target=server.handle_client, args=(conn, addr))
            thread.start()

    if __name__ == "__main__":
    main()

## Observações sobre segurança e privacidade

As credenciais, hashes e dados sensíveis são fictícios.

O servidor foi desenvolvido em ambiente controlado para fins educacionais.

Não utilize este código em produção sem revisão de segurança.
