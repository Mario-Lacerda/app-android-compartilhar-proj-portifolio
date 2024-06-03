# Desafio Dio - Criando um App Android para Compartilhar seu Portfólio de Projetos



### **Projeto: Criando um App Android para Compartilhar seu Portfólio de Projetos**



Este projeto tem como objetivo criar um aplicativo Android completo que permita que você compartilhe seu portfólio de projetos com outras pessoas. O aplicativo incluirá uma lista de seus projetos, com cada projeto exibindo seu nome, descrição, URL e imagem. O aplicativo também incluirá um botão para adicionar novos projetos ao seu portfólio e um botão para editar ou excluir projetos existentes.



#### **Estrutura do Projeto**

O projeto será estruturado em vários módulos, incluindo:

- Módulo `app`: Contém a lógica principal do aplicativo, como navegação e gerenciamento de dados.
- Módulo `core`: Contém classes e utilitários comuns usados por outros módulos.



#### **Recursos do Aplicativo**

O aplicativo terá os seguintes recursos:

- **Tela Principal:** Exibe uma lista de projetos no seu portfólio.
- **Tela de Detalhes do Projeto:** Exibe informações detalhadas sobre um projeto específico, incluindo nome, descrição, URL e imagem.
- **Tela de Adicionar Projeto:** Permite que você adicione um novo projeto ao seu portfólio.
- **Tela de Editar Projeto:** Permite que você edite um projeto existente em seu portfólio.



**Código do Projeto**

#### **MainActivity.kt**

kotlin

```kotlin
package com.example.portfolioapp

import android.content.Intent
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken
import java.io.File

class MainActivity : AppCompatActivity() {

    private lateinit var projectsRecyclerView: RecyclerView
    private lateinit var projectsAdapter: ProjectsAdapter
    private lateinit var projects: List<Project>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        projectsRecyclerView = findViewById(R.id.projectsRecyclerView)
        projectsRecyclerView.layoutManager = LinearLayoutManager(this)

        // Carregar projetos do arquivo JSON
        val projectsFile = File(filesDir, "projects.json")
        if (projectsFile.exists()) {
            val projectsJson = projectsFile.readText()
            val projectsType = object : TypeToken<List<Project>>() {}.type
            projects = Gson().fromJson(projectsJson, projectsType)
        } else {
            projects = listOf()
        }

        projectsAdapter = ProjectsAdapter(projects) { project ->
            val intent = Intent(this, EditProjectActivity::class.java)
            intent.putExtra(EXTRA_PROJECT, project)
            startActivityForResult(intent, EDIT_PROJECT_REQUEST_CODE)
        }
        projectsRecyclerView.adapter = projectsAdapter

        // Botão para adicionar novo projeto
        findViewById<View>(R.id.addProjectButton).setOnClickListener {
            val intent = Intent(this, AddProjectActivity::class.java)
            startActivityForResult(intent, ADD_PROJECT_REQUEST_CODE)
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        if (requestCode == ADD_PROJECT_REQUEST_CODE && resultCode == RESULT_OK) {
            val project = data?.getParcelableExtra<Project>(EXTRA_PROJECT)
            if (project != null) {
                projects = projects + project
                projectsAdapter.notifyItemInserted(projects.size - 1)

                // Salvar projetos no arquivo JSON
                val projectsFile = File(filesDir, "projects.json")
                val projectsJson = Gson().toJson(projects)
                projectsFile.writeText(projectsJson)
            }
        } else if (requestCode == EDIT_PROJECT_REQUEST_CODE && resultCode == RESULT_OK) {
            val project = data?.getParcelableExtra<Project>(EXTRA_PROJECT)
            if (project != null) {
                val index = projects.indexOfFirst { it.id == project.id }
                if (index != -1) {
                    projects[index] = project
                    projectsAdapter.notifyItemChanged(index)

                    // Salvar projetos no arquivo JSON
                    val projectsFile = File(filesDir, "projects.json")
                    val projectsJson = Gson().toJson(projects)
                    projectsFile.writeText(projectsJson)
                }
            }
        }
    }

    companion object {
        private const val ADD_PROJECT_REQUEST_CODE = 1
        private const val EDIT_PROJECT_REQUEST_CODE = 2
        private const val EXTRA_PROJECT = "project"
    }
}
```



#### **ProjectsAdapter.kt**

kotlin

```kotlin
package com.example.portfolioapp

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide

class ProjectsAdapter(private val projects: List<Project>, private val onEditClick: (Project) -> Unit) : RecyclerView.Adapter<ProjectsAdapter.ViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_project, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val project = projects[position]

        holder.projectNameTextView.text = project.name
        holder.projectDescriptionTextView.text = project.description
        holder.projectUrlTextView.text = project.url

        Glide.with(holder.itemView.context)
            .load(project.image)
            .into(holder.projectImageView)

        holder.editButton.setOnClickListener {
            onEditClick(project)
        }
    }

    override fun getItemCount(): Int {
        return projects.size
    }

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val projectNameTextView: TextView = itemView.findViewById(R.id.projectNameTextView)
        val projectDescriptionTextView: TextView = itemView.findViewById(R.id.projectDescriptionTextView)
        val projectUrlTextView: TextView = itemView.findViewById(R.id.projectUrlTextView)
        val projectImageView: ImageView = itemView.findViewById(R.id.projectImageView)
        val editButton: View = itemView.findViewById(R.id.editButton)
    }
}
```



#### **Project.kt**

kotlin

```kotlin
package com.example.portfolioapp

import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class Project(
    val id: Int,
    val name: String,
    val description: String,
    val url: String,
    val image: String
) : Parcelable
```



#### **activity_main.xml**

xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/projectsRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/addProjectButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:layout_gravity="bottom|end"
        android:src="@drawable/ic_add"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```



#### **item_project.xml**

xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/projectImageView"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="16dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/projectNameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:text="Nome do Projeto"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/projectImageView"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/projectDescriptionTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="16dp"
        android:text="Descrição do Projeto"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/projectImageView"
        app:layout_constraintTop_toBottomOf="@+id/projectNameTextView" />

    <TextView
        android:id="@+id/projectUrlTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="16dp"
        android:layout_marginBottom="16dp"
        android:text="URL do Projeto"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/projectImageView"
        app:layout_constraintTop_toBottomOf="@+id/projectDescriptionTextView" />

    <ImageButton
        android:id="@+id/editButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:layout_marginBottom="16dp"
        android:src="@drawable/ic_edit"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```



### **Conclusão**

Este projeto fornece um modelo completo para criar um aplicativo Android para compartilhar seu portfólio de projetos. O aplicativo pode ser personalizado para atender às suas necessidades específicas, adicionando recursos adicionais, como a capacidade de excluir projetos.











# desafio-github-search

Criando um App Android para compartilhar seu portfolio de projeto 

Criar um App Android simples que armazene um usuário do GitHub (informado em uma tela inicial) e liste todos os seus repositórios públicos. Garanta que o nome do usuário seja salvo e o App tenha a capacidade de redefinir essa informação.

![image](https://user-images.githubusercontent.com/5827265/188474294-4472bcc0-24ee-4ccd-80a8-7cee0372e7fa.png)









