# Flask


## Ambiente de Desenvolvimento

Criar uma máquina virtual, para isolar o projeto e não instalar as dependência no Python da raiz.
    
    python -m venv <nome_da_venv>
    
Ativar a máquina virtual para uso.

    <nome_da_venv>/Scripts/activate
    
Atualizar a versão do pip. Só se aparecer a mensagem "WARNING: You are using pip version ...".

    python -m pip install --upgrade pip


## Dependências

    pip install python-dotenv
    pip install Flask
    pip install Flask-SQLAlchemy
    pip install Flask-Migrate

## Variáveis de Ambiente

`.env`

```shell
FLASK_APP = 'main.py'
FLASK_ENV = 'development'
```

`main.py`

```python
from os import getenv
from datetime import datetime

from dotenv import load_dotenv
from flask import Flask, abort, flash, jsonify, redirect, render_template, url_for, request
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_wtf import CSRFProtect


load_dotenv()


app = Flask(__name__)
app.config['SECRET_KEY'] = 'crieUmaSenhaForteNecessarioParaDb&Token'

app.config['SQLALCHEMY_DATABASE_URI'] = getenv('SQLALCHEMY_DATABASE_URI')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

csrf = CSRFProtect(app)


class Pessoa(db.Model):
    __tablename__ = 'pessoas'

    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(40), unique=True, nullable=False)
    idade = db.Column(db.Integer, nullable=False)
    peso = db.Column(db.Float(3,2), nullable=False)
    criado_em = db.Column(db.DateTime, default=datetime.now)

    def __init__(self, nome, idade, peso):
        self.nome = nome
        self.idade = idade
        self.peso = peso


class Endereco(db.Model):
    __tablename__ = 'enderecos'

    id = db.Column(db.Integer, primary_key=True)
    pessoa_id = db.Column(db.Integer, db.ForeignKey('pessoas.id'), nullable=False)
    cep = db.Column(db.String(8), unique=True, nullable=False)
    numero = db.Column(db.String(5), nullable=False)
    criado_em = db.Column(db.DateTime, default=datetime.now)

    def __init__(self, pessoa_id, cep, numero):
        self.pessoa_id = pessoa_id
        self.cep = cep
        self.numero = numero


Migrate(app, db)


@app.route('/')
def index():
    return redirect(url_for('pagina_inicial'))


@app.route('/python-flask')
def pagina_inicial():
    return render_template('pagina-inicial.html', title='Página Inicial')


@app.route('/python-flask/formulario-pessoa')
def formulario_pessoa():
    alterar = request.args.get('alterar', None)
    return render_template('pagina-inicial.html', title='Formulário Pessoa', alterar=alterar)


@app.route('/python-flask/tabela-pessoas')
def tabela_pessoas():
    pessoas = [
        {
            'id': pessoa.id,
            'nome': pessoa.nome,
            'idade': pessoa.idade,
            'peso': round(pessoa.peso),
            'uri': url_for('buscar_pessoa', id=pessoa.id)
        }
        for pessoa in db.session.query(Pessoa).all()
    ]
    return render_template('pagina-inicial.html', title='Tabela Pessoas', pessoas=pessoas)


@app.route('/python-flask/api/pessoa', methods=['POST'])
def criar_pessoa():
    nome, idade, peso = request.form.values()
    pessoa = Pessoa(nome.strip().lower(), idade, peso)
    db.session.add(pessoa)
    db.session.commit()
    pessoa = {
        'id': pessoa.id,
        'nome': pessoa.nome,
        'idade': pessoa.idade,
        'peso': round(pessoa.peso)
    }
    return jsonify(pessoa), 201


@app.route('/python-flask/api/pessoa/<int:id>', methods=['GET'])
def buscar_pessoa(id=None):
    pessoa = db.session.query(Pessoa).get(id)
    if not pessoa:
        abort(404)
    pessoa = {
        'id': pessoa.id,
        'nome': pessoa.nome,
        'idade': pessoa.idade,
        'peso': round(pessoa.peso),
    }
    return jsonify(pessoa)


@app.route('/python-flask/api/pessoas', methods=['GET'])
def buscar_pessoas():
    pessoas = [
        {
            'id': pessoa.id,
            'nome': pessoa.nome,
            'idade': pessoa.idade,
            'peso': round(pessoa.peso),
            'uri': url_for('buscar_pessoa', id=pessoa.id)
        }
        for pessoa in db.session.query(Pessoa).all()
    ]
    return jsonify(pessoas)


@app.route('/python-flask/api/pessoa/<int:id>', methods=['PUT'])
def alterar_pessoa(id=None):
    nome, idade, peso = request.get_json().values()
    pessoa = db.session.query(Pessoa).get(id)
    if not pessoa:
        abort(404)
    pessoa.nome = nome.strip().lower()
    pessoa.idade = idade
    pessoa.peso = peso
    db.session.commit()
    pessoa = {
        'id': pessoa.id,
        'nome': pessoa.nome,
        'idade': pessoa.idade,
        'peso': round(pessoa.peso),
    }
    return jsonify(pessoa)


@app.route('/python-flask/api/pessoa/<int:id>', methods=['DELETE'])
def apagar_pessoa(id=None):
    pessoa = db.session.query(Pessoa).get(id)
    if not pessoa:
        abort(404)
    db.session.delete(pessoa)
    db.session.commit()
    pessoa = {
        'id': pessoa.id,
        'nome': pessoa.nome,
        'idade': pessoa.idade,
        'peso': round(pessoa.peso),
    }
    return jsonify(pessoa)
```

`tabela.js`

```javascript
// Pegue todos os elementos HTML com classe botao e percorra por cada um deles
document.querySelectorAll('.botao').forEach(function(botao) {

    // Observe os elementos e quando ocorrer clique aplique a função definida
    botao.addEventListener('click', function(evento) {

        // Para o evento padrão do browser após o clique
        evento.preventDefault();

        const url = botao.value;

        // Objeto necessário para realizar requições fecth diferentes de GET
        const parametros = {method: botao.name.toUpperCase()};

        if (parametros.method === 'PUT') {
            // JSON.stringify(): transforma um objeto em um JSON válido
            parametros.body = JSON.stringify({
                // prompt(): método que cria um alerta com input
                nome: prompt('nome'),  
                idade: prompt('idade'),
                peso: prompt('peso')
            })

            // Necessário para requisções onde vão ser enviados dados
            parametros.headers = {
                'Content-Type': 'application/json',
                'X-Requested-With': 'XMLHttpRequest',
                'X-CSRFToken': document.querySelector('[name="csrf_token"]').value
            }
        }

        // Método nativo do JS que realiza requição AJAX, parâmetros é opcional
        fetch(url, parametros)
            .then(function(resposta) {
                if (resposta.status == 200) {
                    // Recarrega a página
                    document.location.reload(true);
                }
            })
    })
});
```
    
    
    flask db init
    flask db migrate
    flask db upgrade
    flask run
