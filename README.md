
# Como criar um textarea em Ionic 5 ou Angular que aceite formatações e imagens
Para conseguir resolver este problema eu utilizei uma biblioteca criada pela Microsoft chamada rooster que é open source e pode ser acessada no link abaixo.

Created by Microsoft: [https://github.com/Microsoft/roosterjs](https://github.com/Microsoft/roosterjs).

Eu precisava usar essa Lib no Ionic 5, como o Ionic se baseia no angular eu procurei uma lib já pronta, acabei encontrando este GitHub [https://github.com/insurance-technologies/ng-rooster](https://github.com/insurance-technologies/ng-rooster) que foi extremamente útil para mim.

## Como instalar o rooster

    npm i @instechnologies/ng-rooster
## Como usar
A primeira coisa a fazer depois de instalar a biblioteca é adicionar o módulo a seu **app.module.ts** ou **outro módulo**

### Importando no módulo

```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { FormsModule } from '@angular/forms';
    
    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { NgRoosterModule } from '@instechnologies/ng-rooster';
    
    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        NgRoosterModule, //import the NgRoosterModule
        FormsModule
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
```

### Usando o componente

    <rooster-editor-box style="width: 500px; height: 500px"></rooster-editor-box>

Você pode usar formatações colocando o componente em negrito, itálico, sublinhado, ou até mesmo adicionar as imagens.

As imagens adicionadas são convertidas para stringBase64, assim fazendo parte do seu texto.

Para você usar os eventos de negrito, itálico e etc, é necessário disparar eventos para seu editor, veja:

**component:**
 

     bold$ = new Subject<void>()



**template:**

       <button (click)="bold$.next(0)">Bold</button>
      
       <rooster-editor-box [toggleBold$]="bold$"></rooster-editor-box>

Para obter o resultado final do seu editor, você pode usar o ngModel ou o formControlName.

    <rooster-editor-box [(ngModel)]="content" ></rooster-editor-box>

## Veja como ficou meu código

**template**
```html
<ion-header>
  <ion-toolbar color="secondary">
    <ion-buttons slot="start">
      <ion-back-button></ion-back-button>
      <ion-menu-button></ion-menu-button>
    </ion-buttons>
    <ion-label>TICKET NOVO</ion-label>
  </ion-toolbar>
</ion-header>

<ion-content>
  <ion-list lines="none">
    <ion-item>
      <ion-text>
        <h1>Adicionar Ticket</h1>
      </ion-text>
    </ion-item>
  </ion-list>

  <form [formGroup]="formGroup">
    <ion-list>
      <ion-item>
        <ion-label color="primary" position="stacked">Módulo</ion-label>
        <ion-select formControlName="idModulo" placeholder="Selecione o Módulo" interface="action-sheet" mode="ios">
          <ion-select-option [value]="modulo.id" *ngFor="let modulo of moduloCollection">{{modulo.nome}}
          </ion-select-option>
        </ion-select>
      </ion-item>
      <ion-item>
        <ion-label position="stacked">Versão:</ion-label>
        <ion-input formControlName="versao" placeholder="2020.10.1" type="text">
        </ion-input>
      </ion-item>
      <ion-item>
        <ion-label position="stacked">Assunto:</ion-label>
        <ion-input autofocus="true" formControlName="assunto" placeholder="Informe o assunto" type="text">
        </ion-input>
      </ion-item>
      <ion-item>
        <ion-label position="stacked">Descrição:</ion-label>
        <br>
        <div style="display: flex;">
          <button (click)="bold$.next(0)"><b>B</b></button>
          <button (click)="italic$.next(0)"><b><i>I</i></b></button>
          <button (click)="underline$.next(0)"><b>U</b></button>
        </div>

        <rooster-editor-box formControlName="descricao"
          style="width: 100%; height: 200px; overflow-y: scroll; border: 1px solid gray; padding-inline: 5px;"
          [toggleBold$]="bold$" (isBoldChange)="isBold = $event" [toggleUnderline$]="underline$"
          [toggleItalic$]="italic$"></rooster-editor-box>
      </ion-item>
      <ion-item>
        <ion-label position="stacked">Anexo:</ion-label>
        <input type="file" ng2FileSelect [uploader]="uploader" multiple>
      </ion-item>

      <ion-item *ngFor="let item of uploader.queue">
        <ion-label>
          {{ item?.file?.name }}
        </ion-label>
        <ion-icon name="trash-outline" slot="end" (click)="removerFile(item)"></ion-icon>
      </ion-item>
    </ion-list>

    <ion-button color="secondary" expand="block" [disabled]="!formGroup.valid" (click)="adicionar()">
      Cadastrar
    </ion-button>

  </form>
</ion-content>

<ion-footer>
  <ion-progress-bar type="indeterminate" *ngIf="loading"></ion-progress-bar>
  <ion-toolbar>
    <div class="ion-text-center">
      <img src="../assets/logo.png" style="width: 52px;">
    </div>
  </ion-toolbar>
</ion-footer>
```

**component**
```javascript
import { Component, ViewChild, ElementRef, Directive, ViewChildren, QueryList } from '@angular/core';
import { FormGroup, FormBuilder, FormControl, Validators } from '@angular/forms';
import { UtilService } from 'src/app/services/util.service';
import { NavController } from '@ionic/angular';
import { TicketService } from 'src/app/services/ticket.service';
import { FileUploader, FileLikeObject } from 'ng2-file-upload';
import { ModuloService } from 'src/app/services/modulo.service';
import { EnumTipoUsuario } from 'src/app/enums/enum-tipo-usuario.enum';
import { Subject } from 'rxjs';

@Component({
  selector: 'app-ticket-novo',
  templateUrl: './ticket-novo.page.html',
  styleUrls: ['./ticket-novo.page.scss'],
})

export class TicketNovoPage{
  
  public formGroup: FormGroup;
  loading: boolean = false;
  

  public uploader: FileUploader = new FileUploader({});
  public hasBaseDropZoneOver: boolean = false;
  moduloCollection: any[] = [];

  bold$ = new Subject<void>();
  italic$ = new Subject<void>();
  underline$ = new Subject<void>();

  constructor(public formBuilder: FormBuilder, private utilService: UtilService, private navCtrl: NavController, private ticketService : TicketService, private moduloService : ModuloService) {
    this.formGroup = formBuilder.group({

      assunto: new FormControl('', Validators.compose([
        Validators.required,
        Validators.maxLength(150),
        Validators.minLength(1),
      ])),
      descricao: new FormControl('', Validators.compose([
        Validators.required,
        //Validators.maxLength(8000),
        Validators.minLength(1),
      ])),
      idModulo: new FormControl('', Validators.compose([
        Validators.required,
      ])),
      versao: new FormControl('', Validators.compose([
      ])),
    });
    this.loadModulo();
  }

  getFiles(): FileLikeObject[] {
    return this.uploader.queue.map((fileItem) => {
      return fileItem.file;
    });
  }

  adicionar() {
    let files = this.getFiles();
    //Verifica se excedeu o tamanho do arquivo

    let tamanhoOk: boolean = true;
    let tamanhoArquivoAceito = 1024 * 10; //10MB
    files.forEach((file) => {
      let tamanhoDoArquivo = file.size / 1024;
      if (tamanhoDoArquivo > tamanhoArquivoAceito) {
        tamanhoOk = false;
      }
    });

    
    let formData = new FormData();
    formData.append('assunto', this.formGroup.value.assunto);
    formData.append('descricao', this.formGroup.value.descricao);
    formData.append('idModulo', this.formGroup.value.idModulo);
    formData.append('versao', this.formGroup.value.versao);
    
    files.forEach((file) => {
      formData.append('files[]', file.rawFile, file.name);
    });
    console.log(formData);
    //debugger;
    
    this.utilService.showLoading();
    this.loading = true;
    this.ticketService.adicionar(formData)
      .then((response: any) => {
        this.utilService.hideLoading();
        this.loading = false;
        if (response.success == true) {
          this.utilService.showAlert("Operação realizada com sucesso", ()=>{
            this.uploader.clearQueue();
            this.navCtrl.navigateRoot('cliente/ticket-consultar');
          });
        }
        else {
          this.utilService.hideLoading();
          this.utilService.showError(response);
        }
      })
      .catch((error) => {
        this.loading = false;
        
        this.utilService.showAlert("Desculpe, operação falhou! Tente novamente mais tarde.");
      });
  }
  removerFile(item){
    this.uploader.removeFromQueue(item);
  }
  loadModulo() {
    this.loading = true;

    this.moduloService.listar(EnumTipoUsuario.Cliente)
      .then((response : any) => {
        this.loading = false;
        this.moduloCollection = response.data;
      })
      .catch((erro) => {
        this.loading = false        ;
        this.utilService.showAlert("Desculpe, operação falhou! Tente novamente mais tarde.");
        console.error(erro);
      });
  }
}
```

**module**
```javascript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

import { IonicModule } from '@ionic/angular';

import { TicketNovoPageRoutingModule } from './ticket-novo-routing.module';

import { TicketNovoPage } from './ticket-novo.page';
import { FileUploadModule } from 'ng2-file-upload';
import { NgRoosterModule } from '@instechnologies/ng-rooster';

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    FileUploadModule,
    IonicModule,
    NgRoosterModule, //import the NgRoosterModule
    TicketNovoPageRoutingModule
  ],
  declarations: [TicketNovoPage]
})
export class TicketNovoPageModule {}

```

### Funcionalidade em ação
![](https://github.com/pauloanalista/Como-criar-um-TextArea-em-Ionic-5-que-aceite-formatacoes-e-imagens/blob/main/rooster.gif)


# VEJA TAMBÉM
## Grupo de Estudo no Telegram
- [Participe gratuitamente do grupo de estudo](https://t.me/blogilovecode)

## Cursos baratos!
- [Meus cursos na Udemy](https://olha.la/udemy)
- [Outros cursos](https://olha.la/cursos)

## Fique ligado, acesse!
- [Blog ILoveCode](https://ilovecode.com.br)

## Novidades, cupons de descontos e cursos gratuitos
https://olha.la/ilovecode-receber-cupons-novidades
