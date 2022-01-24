# FLASK


## AMBIENTE DE DESENVOLVIMENTO

Criar uma máquina virtual, para isolar o projeto e não instalar as dependência no Python da raiz.
    
    python -m venv <nome_da_venv>
    
Ativar a máquina virtual para uso.

    <nome_da_venv>/Scripts/activate
    
Atualizar a versão do pip. Só se aparecer a mensagem "WARNING: You are using pip version ...".

    python -m pip install --upgrade pip


## DEPENDÊNCIAS

    pip install python-dotenv
    pip install Flask
    pip install Flask-SQLAlchemy
    pip install Flask-Migrate

## VARIÁVEIS DE AMBIENTE

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
from flask import Flask, flash, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate


load_dotenv()


app = Flask(__name__)
app.config['SECRET_KEY'] = 'crieUmaSenhaForteNecessarioParaDb&Token'

app.config['SQLALCHEMY_DATABASE_URI'] = getenv('SQLALCHEMY_DATABASE_URI')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)


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


Migrate(app, db)


@app.route('/')
def index():
    return 'Página índice.'


@app.route('/pessoa/<nome>/<int:idade>/<float:peso>')
def pessoa(nome=None, idade=None, peso=None):
    if nome and idade and peso:
        nome = nome.strip().lower()

        try:
            db.session.add(Pessoa(nome=nome, idade=idade, peso=peso))
            db.session.commit()
            flash('Registro salvo com sucesso.', 'success')
        except:
            db.session.rollback()
            flash('Erro ao tentar salvar no banco de dados.', 'danger')
        finally:
            return redirect(url_for('index'))

    return 'URL inválida'
```
    
    
    flask db init
    flask db migrate
    flask db upgrade
    flask run
