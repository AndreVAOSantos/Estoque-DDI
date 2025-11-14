# Usar a imagem oficial do Python 3.11
FROM python:3.11-slim

# Definir o diretório de trabalho no container
WORKDIR /app

# Copiar o arquivo de dependências
COPY requirements.txt .

# Instalar as dependências
RUN pip install --no-cache-dir -r requirements.txt

# Copiar o código-fonte da aplicação
COPY src/ ./src/

# Garantir que o output do Python não seja bufferizado
ENV PYTHONUNBUFFERED=1

# Comando para executar a aplicação
CMD ["python", "src/app.py"]
