import 'package:flutter/material.dart';
import 'package:intl/intl.dart'; // Pacote permitido para formatação de data
import 'dart:async'; // Necessário para Future.delayed

void main() {
  runApp(const AplicativoNotas());
}

// Modelo de dados para uma única nota
class Nota {
  String titulo; // Campo para o título da nota, agora mutável
  String conteudo; // Campo para o conteúdo da nota, agora mutável
  final DateTime timestamp; // Timestamp permanece final para a criação inicial

  Nota({required this.titulo, required this.conteudo, required this.timestamp});
}

class AplicativoNotas extends StatelessWidget {
  const AplicativoNotas({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bloco de Notas',
      theme: ThemeData.dark(),
      home: const TelaNotas(), // widget inicial da home
      debugShowCheckedModeBanner: false,
    );
  }
}

// Tela principal para exibir e gerenciar notas
class TelaNotas extends StatefulWidget {
  const TelaNotas({super.key});

  @override
  _TelaNotasState createState() => _TelaNotasState();
}

class _TelaNotasState extends State<TelaNotas> {
  // Lista de notas
  final List<Nota> _notas = <Nota>[];

  // Controladores dos campos de texto
  final TextEditingController _controladorTitulo =
      TextEditingController(); // Controlador para o título
  final TextEditingController _controladorConteudo =
      TextEditingController(); // Controlador para o conteúdo

  // Variável para controlar o índice da nota que está sendo editada
  // Null se nenhuma nota estiver sendo editada, caso contrário, o índice da nota.
  int? _indiceNotaEmEdicao;

  // Função para salvar uma nota
  void _salvarNota() {
    setState(() {
      if (_controladorTitulo.text.isNotEmpty && _controladorConteudo.text.isNotEmpty) {
        if (_indiceNotaEmEdicao != null) {
          // Se estamos editando uma nota existente
          _notas[_indiceNotaEmEdicao!].titulo = _controladorTitulo.text;
          _notas[_indiceNotaEmEdicao!].conteudo = _controladorConteudo.text;
          _indiceNotaEmEdicao = null; // Sair do modo de edição
        } else {
          // Se estamos adicionando uma nova nota
          _notas.add(
            Nota(
              titulo: _controladorTitulo.text,
              conteudo: _controladorConteudo.text,
              timestamp: DateTime.now(),
            ),
          );
        }
        _controladorTitulo.clear();
        _controladorConteudo.clear();
      }
    });
  }

  // Função para iniciar a edição de uma nota
  void _editarNota(int index) {
    setState(() {
      _indiceNotaEmEdicao = index;
      _controladorTitulo.text = _notas[index].titulo;
      _controladorConteudo.text = _notas[index].conteudo;
    });
  }

  // Função para remover uma nota
  void _removerNota(int index) {
    setState(() {
      // Se a nota a ser removida é a que está sendo editada, cancelar a edição
      if (_indiceNotaEmEdicao == index) {
        _indiceNotaEmEdicao = null;
        _controladorTitulo.clear();
        _controladorConteudo.clear();
      } else if (_indiceNotaEmEdicao != null && index < _indiceNotaEmEdicao!) {
        // Se uma nota anterior à que está sendo editada for removida, ajustar o índice
        _indiceNotaEmEdicao = _indiceNotaEmEdicao! - 1;
      }
      _notas.removeAt(index);
    });
  }

  // Função para cancelar a edição
  void _cancelarEdicao() {
    setState(() {
      _indiceNotaEmEdicao = null;
      _controladorTitulo.clear();
      _controladorConteudo.clear();
    });
  }

  @override
  void dispose() {
    _controladorTitulo.dispose();
    _controladorConteudo.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Bloco de Notas'),
      ),
      body: Column(
        children: <Widget>[
          // Campos de entrada e botão de salvar/adicionar
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              children: <Widget>[
                TextField(
                  controller: _controladorTitulo,
                  decoration: InputDecoration(
                    hintText: 'Título da nota',
                    border: const OutlineInputBorder(),
                    // Botão para cancelar a edição, visível apenas no modo de edição
                    suffixIcon: _indiceNotaEmEdicao != null
                        ? IconButton(
                            icon: const Icon(Icons.cancel),
                            onPressed: _cancelarEdicao,
                            tooltip: 'Cancelar edição',
                          )
                        : null,
                  ),
                ),
                const SizedBox(height: 8.0),
                Row(
                  children: <Widget>[
                    Expanded(
                      child: TextField(
                        controller: _controladorConteudo,
                        decoration: const InputDecoration(
                          hintText: 'Conteúdo da nota',
                          border: OutlineInputBorder(),
                        ),
                        maxLines: null, // Permite múltiplas linhas para o conteúdo
                        keyboardType: TextInputType.multiline, // Tipo de teclado para entrada de múltiplas linhas
                      ),
                    ),
                    IconButton(
                      // Altera o ícone com base no modo de edição
                      icon: Icon(
                          _indiceNotaEmEdicao != null ? Icons.save : Icons.add),
                      onPressed: _salvarNota,
                      tooltip: _indiceNotaEmEdicao != null
                          ? 'Salvar nota'
                          : 'Adicionar nota',
                    ),
                  ],
                ),
              ],
            ),
          ),

          // Lista de notas
          Expanded(
            child: ListView.builder(
              itemCount: _notas.length,
              itemBuilder: (BuildContext context, int index) {
                final Nota nota = _notas[index];
                // Formata o carimbo de data/hora para exibir dia e hora
                final String timestampFormatado =
                    DateFormat('dd/MM/yyyy HH:mm').format(nota.timestamp);

                return Card(
                  margin: const EdgeInsets.symmetric(
                      horizontal: 8.0, vertical: 4.0),
                  child: ListTile(
                    title: Text(
                      nota.titulo, // Exibe o título da nota
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Text(nota.conteudo), // Exibe o conteúdo da nota
                        const SizedBox(height: 4.0),
                        Text(
                          timestampFormatado, // Exibe o carimbo de data/hora formatado
                          style: const TextStyle(
                              color: Colors.white70, fontSize: 12.0),
                        ),
                      ],
                    ),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min, // Ocupa o mínimo de espaço
                      children: <Widget>[
                        IconButton(
                          icon: const Icon(Icons.edit),
                          onPressed: () => _editarNota(index),
                          tooltip: 'Editar nota',
                        ),
                        IconButton(
                          icon: const Icon(Icons.delete),
                          onPressed: () => _removerNota(index),
                          tooltip: 'Remover nota',
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
