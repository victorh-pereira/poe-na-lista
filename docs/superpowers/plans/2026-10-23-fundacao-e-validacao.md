# Põe na Lista — Plano 1: Fundação e validação técnica

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ter um app Android em Kotlin/Compose rodando no Galaxy A54 que autentica no Supabase, grava um produto no Room e envia/baixa uma foto — provando a stack antes de construir o produto.

**Architecture:** Módulo único, pacotes por feature, UI em Compose observando ViewModels que falam com repositórios. Room é a fonte da verdade local; o Supabase fica atrás de interfaces (`AuthGateway`, `PhotoRemote`) para poder ser substituído. Sem Hilt, sem CameraX, sem sincronização nesta etapa.

**Tech Stack:** Kotlin 2.x, Jetpack Compose + Material 3, Navigation Compose, Room + KSP, DataStore, Coil 3, `supabase-kt` (auth/storage/postgrest) sobre Ktor OkHttp, kotlinx.serialization, JUnit4 + Turbine + coroutines-test.

**Spec:** `docs/superpowers/specs/2026-10-23-poe-na-lista-design.md`

**Este é o Plano 1 de 5.** Ele implementa apenas a Etapa 0 da spec (validação técnica). Os planos seguintes — catálogo, compra, sincronização e polimento — serão escritos depois que este rodar no aparelho, porque as interfaces reais (versões do `supabase-kt`, comportamento do Room, compressão) mudam decisões dos planos seguintes. Cada plano produz software instalável e testável por si.

## Global Constraints

- Package: `com.victorh.poenalista`; nome exibido: `Põe na Lista`.
- `minSdk 26`; `compileSdk`/`targetSdk` = versão estável mais recente no momento da criação.
- Idioma da UI: pt-BR. Formatação `R$ 1.234,56` e data `dd/MM/yyyy`.
- Identificadores de código em inglês; textos de UI em português, em `strings.xml`.
- Dinheiro em **centavos inteiros** (`Long`). Peso em **gramas** (`Int`). Nunca `Float`/`Double` para dinheiro.
- Sem Hilt. Sem CameraX (usar `ActivityResultContracts.TakePicture`).
- `supabase-kt` só é importado em `core/auth` e `core/images`; o resto do app depende das interfaces.
- Módulo único; pacotes por feature. Sem multi-módulo nesta fase.
- Commits convencionais (`feat:`, `fix:`, `test:`, `chore:`, `docs:`) com descrição em português.
- Nenhum segredo no repositório: `local.properties`, `*.jks`, `*.keystore` no `.gitignore`.
- Testes de lógica em `test/` (JVM); Room, Compose e Bitmap em `androidTest/` (aparelho conectado).
- Rodar `gradlew.bat testDebugUnitTest` e `gradlew.bat connectedDebugAndroidTest` antes de cada commit que toque código.

## Review Focus

1. **Primeiro login sem internet** — o app deve mostrar "sem conexão" e continuar utilizável, sem crash nem sessão parcial.
2. **Câmera cancelada ou permissão negada** — nenhum arquivo órfão, nenhum crash, o formulário permanece preenchido.
3. **Processo morto durante o salvamento** — Room transacional: ou o produto existe completo, ou não existe.
4. **Entrada de dinheiro hostil** — `"1.234,56"`, `"0,05"`, `""`, `"abc"`, `"-1"`: parse correto ou erro claro; nunca valor silenciosamente errado.
5. **Sessão expirada entre aberturas** — refresh automático ou retorno ao login, sem apagar os dados locais.

---

## Pré-requisitos

### Task 0: Ambiente de desenvolvimento (sem código)

**Files:** nenhum (configuração da máquina).

**Interfaces:**
- Consumes: nada.
- Produces: Android Studio, SDK, `adb` e o A54 autorizado — necessários para todas as tasks.

- [ ] **Step 1: Instalar o Android Studio**

Baixe a versão estável em `developer.android.com/studio` e instale com as opções padrão. Ele traz o JDK e o SDK. Nesta máquina não há Android Studio, SDK nem `adb`; o `java` do PATH é o 8 e **não serve**.

- [ ] **Step 2: Instalar os componentes do SDK**

No Android Studio: `Tools → SDK Manager` → instalar a Platform mais recente, `Android SDK Build-Tools`, `Android SDK Platform-Tools` e `Android SDK Command-line Tools`.

- [ ] **Step 3: Configurar o PATH**

Adicione `%LOCALAPPDATA%\Android\Sdk\platform-tools` ao PATH do usuário e reabra o terminal.

- [ ] **Step 4: Preparar o aparelho**

No A54: `Configurações → Sobre o telefone → Informações do software` → tocar 7× em "Número da versão"; depois `Opções do desenvolvedor → Depuração USB` ligada. Conecte por USB e aceite o aviso de autorização.

- [ ] **Step 5: Verificar**

Run: `adb devices` e `java -version`
Expected: o A54 aparece como `device` (não `unauthorized`); Java 17 ou superior (o que o Android Studio usa).

---

### Task 1: Projeto Android mínimo com Compose

**Files:**
- Create: `settings.gradle.kts`, `build.gradle.kts`, `gradle/libs.versions.toml`, `app/build.gradle.kts`
- Create: `app/src/main/AndroidManifest.xml`
- Create: `app/src/main/java/com/victorh/poenalista/MainActivity.kt`
- Create: `app/src/main/res/values/strings.xml`
- Create: `.gitignore`
- Test: `app/src/androidTest/java/com/victorh/poenalista/AppLaunchTest.kt`

**Interfaces:**
- Consumes: nada.
- Produces: `MainActivity` (ComponentActivity) e `AppRoot()` composable, usados pelas tasks seguintes.

- [ ] **Step 1: Criar o projeto**

Use o wizard do Android Studio (Empty Activity, Compose), direto em `D:\Scripts\poenalista`, mantendo a pasta `docs/` existente. Package `com.victorh.poenalista`, `minSdk 26`. Se preferir linha de comando, crie os arquivos Gradle manualmente com as mesmas configurações.

Adicione ao `.gitignore`: `.gradle/`, `build/`, `local.properties`, `*.jks`, `*.keystore`, `.idea/` (exceto `codeStyles`), `*.iml`, `.kotlin/`.

- [ ] **Step 2: Escrever o teste que falha**

`app/src/androidTest/java/com/victorh/poenalista/AppLaunchTest.kt`:

```kotlin
@RunWith(AndroidJUnit4::class)
class AppLaunchTest {
    @get:Rule val rule = createAndroidComposeRule<MainActivity>()

    @Test
    fun showsAppName() {
        rule.onNodeWithText("Põe na Lista").assertIsDisplayed()
    }
}
```

Adicione as dependências de teste: `androidx.compose.ui:ui-test-junit4`, `androidx.test.ext:junit`, `androidx.test.espresso:espresso-core`, e `debugImplementation androidx.compose.ui:ui-test-manifest`.

- [ ] **Step 3: Rodar e ver falhar**

Run: `gradlew.bat connectedDebugAndroidTest`
Expected: FAIL — nenhum nó com o texto "Põe na Lista".

- [ ] **Step 4: Implementar o mínimo**

`MainActivity.kt` com `setContent { AppRoot() }`; `AppRoot()` com `Surface` e `Text(stringResource(R.string.app_name))`. `strings.xml` com `<string name="app_name">Põe na Lista</string>`.

- [ ] **Step 5: Rodar e ver passar**

Run: `gradlew.bat connectedDebugAndroidTest`
Expected: PASS, 1 teste.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat: projeto Android com Compose e teste de inicializacao"
```

---

### Task 2: Tema e navegação com três abas

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/ui/theme/Color.kt`, `Type.kt`, `Theme.kt`
- Create: `app/src/main/java/com/victorh/poenalista/ui/navigation/AppNavHost.kt`, `Destinations.kt`
- Modify: `app/src/main/java/com/victorh/poenalista/MainActivity.kt`
- Test: `app/src/androidTest/java/com/victorh/poenalista/NavigationTest.kt`

**Interfaces:**
- Consumes: `AppRoot()` da Task 1.
- Produces: `PoeNaListaTheme(content: @Composable () -> Unit)`; `AppNavHost()`; rotas `Home`, `Products`, `History` em `Destinations`.

- [ ] **Step 1: Escrever o teste que falha**

`NavigationTest`: abre o app, toca em "Produtos", espera ver o texto "Nenhum produto cadastrado"; toca em "Histórico", espera "Nenhuma compra registrada".

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*NavigationTest"`
Expected: FAIL — abas não existem.

- [ ] **Step 3: Implementar**

- `Theme.kt`: `MaterialTheme` com `lightColorScheme()`/`darkColorScheme()`, seguindo o sistema; dynamic color desativado (identidade fixa).
- `Destinations.kt`: `sealed interface Destination { val route: String }` com `Home`, `Products`, `History`.
- `AppNavHost.kt`: `Scaffold` com `NavigationBar` de 3 itens e `NavHost` com uma tela placeholder por rota.
- `MainActivity` passa a chamar `PoeNaListaTheme { AppNavHost() }`.

Adicione `androidx.navigation:navigation-compose`.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*NavigationTest"`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: tema Material 3 e navegacao com tres abas"
```

---

### Task 3: Dinheiro em centavos

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/money/Cents.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/money/MoneyFormat.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/money/MoneyFormatTest.kt`

**Interfaces:**
- Consumes: nada.
- Produces:
  - `@JvmInline value class Cents(val raw: Long)` com `init { require(raw >= 0) }`
  - `fun Cents.times(quantity: Int): Cents`
  - `object MoneyFormat { fun format(cents: Cents): String; fun parse(text: String): Cents? }`

- [ ] **Step 1: Escrever os testes que falham**

```kotlin
class MoneyFormatTest {
    @Test fun formats_brazilian() = assertEquals("R$ 1.234,56", MoneyFormat.format(Cents(123456)))
    @Test fun formats_zero() = assertEquals("R$ 0,00", MoneyFormat.format(Cents(0)))
    @Test fun parses_with_currency_and_thousands() = assertEquals(Cents(123456), MoneyFormat.parse("R$ 1.234,56"))
    @Test fun parses_plain_comma() = assertEquals(Cents(1234), MoneyFormat.parse("12,34"))
    @Test fun parses_cents_only() = assertEquals(Cents(5), MoneyFormat.parse("0,05"))
    @Test fun rejects_empty() = assertNull(MoneyFormat.parse(""))
    @Test fun rejects_letters() = assertNull(MoneyFormat.parse("abc"))
    @Test fun rejects_negative() = assertNull(MoneyFormat.parse("-1,00"))
    @Test fun rejects_three_decimals() = assertNull(MoneyFormat.parse("1,234"))
    @Test fun multiplies_quantity() = assertEquals(Cents(3300), Cents(550).times(6))
    @Test fun rejects_zero_quantity() = assertThrows(IllegalArgumentException::class.java) { Cents(100).times(0) }
}
```

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*MoneyFormatTest"`
Expected: FAIL — classes não existem.

- [ ] **Step 3: Implementar**

`parse` remove `R$`, espaços e pontos de milhar; aceita vírgula como decimal; exige no máximo 2 casas; retorna `null` para qualquer entrada inválida. `format` usa `NumberFormat` pt-BR com 2 casas e prefixo `R$ `. `times` usa `Math.multiplyExact` para detectar overflow.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat testDebugUnitTest --tests "*MoneyFormatTest"`
Expected: PASS, 11 testes.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: dinheiro em centavos com parse e formatacao pt-BR"
```

---

### Task 4: Room com produtos e categorias

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/database/AppDatabase.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/database/ProductEntity.kt`, `CategoryEntity.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/database/ProductDao.kt`, `CategoryDao.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/database/Converters.kt`
- Test: `app/src/androidTest/java/com/victorh/poenalista/core/database/ProductDaoTest.kt`

**Interfaces:**
- Consumes: `Cents` não é usado na entidade; dinheiro fica para a Task 5.
- Produces:
  - `ProductEntity(id, name, categoryId, unit, photoPath, note, active, homeStatus, createdAt, updatedAt)`
  - `CategoryEntity(id, name, sortOrder, active)`
  - `ProductDao`: `observeAll(): Flow<List<ProductEntity>>`, `observeByStatus(status: String)`, `getById(id: String): ProductEntity?`, `upsert(entity): Unit`, `setActive(id, active)`, `setHomeStatus(id, status)`
  - `CategoryDao`: `observeAll()`, `count(): Int`, `upsertAll(list)`
  - `AppDatabase.getInstance(context): AppDatabase`

- [ ] **Step 1: Escrever os testes que falham**

`ProductDaoTest` com `Room.inMemoryDatabaseBuilder`:

```kotlin
@Test fun inserts_and_reads_product() = runTest { ... }
@Test fun observes_only_out_of_stock() = runTest { ... }
@Test fun update_keeps_single_row() = runTest { ... }
@Test fun categories_are_ordered_by_sort_order() = runTest { ... }
@Test fun failed_transaction_rolls_back() = runTest {
    // dentro de db.withTransaction { productDao.upsert(...); error("boom") }
    // após capturar a exceção, productDao.observeAll().first() deve estar vazio
}
```

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*ProductDaoTest"`
Expected: FAIL — banco não existe.

- [ ] **Step 3: Implementar**

Entidades com `@Entity(tableName = "products")` e `@Entity(tableName = "categories")`. `unit` e `homeStatus` como `String`. Timestamps como `Long` (epoch millis). DAOs com `@Query`/`@Upsert`. Índices: `products(active, homeStatus)`, `products(name)`, `categories(sortOrder)`. Adicione `room-runtime`, `room-ktx` e o plugin KSP com `room-compiler`.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*ProductDaoTest"`
Expected: PASS, 5 testes.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: banco local com produtos e categorias"
```

---

### Task 5: Modelo de domínio, validação e contrato do repositório

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/model/Product.kt`, `Category.kt`, `MeasureUnit.kt`, `HomeStatus.kt`, `ProductDraft.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/model/ProductRepository.kt`, `CategoryRepository.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/model/ProductValidation.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/data/ProductMapper.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/model/ProductValidationTest.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/data/ProductMapperTest.kt`

**Interfaces:**
- Consumes: `ProductEntity`, `CategoryEntity` (Task 4).
- Produces:

```kotlin
enum class MeasureUnit { UNIT, KG, GRAM, LITER, MILLILITER, PACK, BOX, DOZEN, TRAY }
enum class HomeStatus { IN_STOCK, OUT }

data class Product(
    val id: String, val name: String, val categoryId: String,
    val unit: MeasureUnit, val photoPath: String?, val note: String?,
    val active: Boolean, val homeStatus: HomeStatus,
    val createdAt: Long, val updatedAt: Long,
)

data class Category(val id: String, val name: String, val sortOrder: Int, val active: Boolean)
data class ProductDraft(val name: String, val categoryId: String, val unit: MeasureUnit, val photoPath: String?, val note: String?)

sealed interface SaveProductResult {
    data class Success(val product: Product) : SaveProductResult
    data class Invalid(val error: ProductValidationError) : SaveProductResult
}
enum class ProductValidationError { EMPTY_NAME, NAME_TOO_LONG, UNKNOWN_CATEGORY }

interface ProductRepository {
    fun observeActive(query: String): Flow<List<Product>>
    suspend fun get(id: String): Product?
    suspend fun save(draft: ProductDraft, id: String? = null): SaveProductResult
    suspend fun setHomeStatus(id: String, status: HomeStatus)
    suspend fun deactivate(id: String)
}
interface CategoryRepository {
    fun observeAll(): Flow<List<Category>>
    suspend fun seedDefaults()
}
```

- [ ] **Step 1: Escrever os testes que falham**

`ProductValidationTest`: nome vazio → `EMPTY_NAME`; nome com 81 caracteres → `NAME_TOO_LONG`; nome com espaços nas pontas é normalizado; categoria vazia → `UNKNOWN_CATEGORY`; draft válido → `null` (sem erro).

`ProductMapperTest`: ida e volta `ProductEntity` ↔ `Product` preserva todos os campos; `MeasureUnit` desconhecida cai em `UNIT` (fallback documentado).

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*ProductValidationTest" --tests "*ProductMapperTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

`ProductValidation.validate(draft, categoryExists): ProductValidationError?` — função pura, recebe um `Boolean` para existência da categoria, para não depender de banco no teste. `ProductMapper` com funções `toDomain()`/`toEntity()`.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat testDebugUnitTest --tests "*ProductValidationTest" --tests "*ProductMapperTest"`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: modelo de produto, validacao e contrato de repositorio"
```

---

### Task 6: Repositório de produtos e seed de categorias

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/data/ProductRepositoryImpl.kt`, `CategoryRepositoryImpl.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/database/DefaultCategories.kt`
- Create: `app/src/main/java/com/victorh/poenalista/app/AppContainer.kt`
- Test: `app/src/androidTest/java/com/victorh/poenalista/core/data/ProductRepositoryTest.kt`

**Interfaces:**
- Consumes: DAOs (Task 4), modelo e validação (Task 5).
- Produces: `ProductRepositoryImpl(productDao, categoryDao, clock: () -> Long)`; `CategoryRepositoryImpl(categoryDao)`; `AppContainer(context)` com `productRepository`, `categoryRepository`, `authRepository`, `photoStore`, `photoRemote` (as duas últimas entram nas Tasks 7–9).

- [ ] **Step 1: Escrever os testes que falham**

`ProductRepositoryTest` (in-memory DB real):
- criar produto válido → aparece em `observeActive("")`;
- criar com categoria inexistente → `Invalid(UNKNOWN_CATEGORY)` e nada gravado;
- editar mantém o mesmo `id` e atualiza `updatedAt`;
- `setHomeStatus(OUT)` reflete no fluxo;
- `deactivate` remove de `observeActive` mas mantém `get(id)`;
- `seedDefaults()` duas vezes não duplica categorias;
- falha no meio de um `save` não deixa produto parcial (transação).

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*ProductRepositoryTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

`DefaultCategories`: Alimentos, Carnes, Hortifruti, Padaria, Frios, Limpeza, Higiene, Bebidas, Pet, Outros (ordem 0–9). `ProductRepositoryImpl` usa `withTransaction` para validar + gravar, gera `UUID.randomUUID().toString()`, normaliza o nome e define timestamps pelo `clock`. `AppContainer` criado no `Application`.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat connectedDebugAndroidTest --tests "*ProductRepositoryTest"`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: repositorio de produtos e categorias padrao"
```

---

### Task 7: Autenticação atrás de interface

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/auth/AuthRepository.kt`, `AuthGateway.kt`, `Session.kt`, `AuthResult.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/auth/AuthRepositoryImpl.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/auth/SupabaseAuthGateway.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/auth/SessionStore.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/auth/AuthRepositoryImplTest.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/auth/FakeAuthGateway.kt`

**Interfaces:**
- Consumes: `AppContainer` (Task 6).
- Produces:

```kotlin
data class Session(val userId: String, val email: String)
sealed interface AuthResult {
    data object Success : AuthResult
    data class Failure(val kind: AuthFailure, val message: String) : AuthResult
}
enum class AuthFailure { INVALID_CREDENTIALS, NO_CONNECTION, UNKNOWN }

interface AuthGateway {
    suspend fun signIn(email: String, password: String): Session
    suspend fun signUp(email: String, password: String): Session
    suspend fun signOut()
    fun currentSession(): Session?
}
interface SessionStore { fun load(): Session?; fun save(session: Session?); }

interface AuthRepository {
    val session: StateFlow<Session?>
    suspend fun signIn(email: String, password: String): AuthResult
    suspend fun signUp(email: String, password: String): AuthResult
    suspend fun signOut()
}
```

- [ ] **Step 1: Escrever os testes que falham**

`AuthRepositoryImplTest` com `FakeAuthGateway`:
- login válido → `Success` e `session` atualizado;
- credencial inválida → `Failure(INVALID_CREDENTIALS)`;
- `IOException`/timeout → `Failure(NO_CONNECTION)` e mensagem "Sem conexão. Tente novamente.";
- sessão existente ao iniciar → `session` já preenchida;
- `signOut` limpa sessão e store.

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*AuthRepositoryImplTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

`AuthRepositoryImpl` mapeia exceções do gateway para `AuthFailure` (nunca deixa exceção vazar para a UI). `SessionStore` em DataStore grava `userId` e `email`. `SupabaseAuthGateway` usa `supabase-kt` (`auth-kt`) com engine OkHttp; `currentSession()` lê a sessão persistida pelo SDK.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat testDebugUnitTest --tests "*AuthRepositoryImplTest"`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: autenticacao com Supabase atras de interface"
```

---

### Task 8: Compressão e armazenamento local de foto

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/images/ImageSizing.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/images/ImageCompressor.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/images/PhotoStore.kt`, `LocalPhotoStore.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/images/ImageSizingTest.kt`
- Test: `app/src/androidTest/java/com/victorh/poenalista/core/images/ImageCompressorTest.kt`

**Interfaces:**
- Consumes: nada.
- Produces:

```kotlin
data class ImageSize(val width: Int, val height: Int)
fun fitWithin(source: ImageSize, maxSide: Int): ImageSize

class ImageCompressor {
    /** Lê [source], aplica orientação EXIF, reduz e grava JPEG em [target]. Retorna o tamanho final. */
    fun compress(source: File, target: File, maxSide: Int = 1080, quality: Int = 80): ImageSize
}

interface PhotoStore {
    /** Copia/comprime [source] para o armazenamento interno. Retorna o caminho local ou null se cancelado. */
    suspend fun save(productId: String, sourcePath: String): String?
    fun pathOf(productId: String): String
    suspend fun delete(productId: String)
}
```

- [ ] **Step 1: Escrever os testes que falham**

`ImageSizingTest` (JVM): 4000×3000 → 1080×810; 3000×4000 → 810×1080; 800×600 → inalterado; `maxSide = 0` lança `IllegalArgumentException`.

`ImageCompressorTest` (androidTest): gera bitmap 3000×2000, comprime, verifica maior lado = 1080, arquivo < 300 KB, JPEG válido; arquivo de origem inexistente → retorna erro tratado, sem crash; `PhotoStore.save` com caminho inválido → `null` e nenhum arquivo órfão.

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*ImageSizingTest"` e `gradlew.bat connectedDebugAndroidTest --tests "*ImageCompressorTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

`fitWithin` com aritmética inteira e `roundToInt`. `ImageCompressor` usa `BitmapFactory` com `inSampleSize`, `ExifInterface` para rotação e `Bitmap.compress(JPEG, quality)`. `LocalPhotoStore` grava em `context.filesDir/fotos/{productId}.jpg` e só mantém o arquivo após compressão bem-sucedida (grava em `.tmp` e renomeia).

- [ ] **Step 4: Rodar e ver passar**

Expected: PASS nos dois comandos.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: compressao e armazenamento local de fotos"
```

---

### Task 9: Upload de foto atrás de interface

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/core/images/PhotoRemote.kt`
- Create: `app/src/main/java/com/victorh/poenalista/core/images/SupabasePhotoRemote.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/images/FakePhotoRemote.kt`
- Test: `app/src/test/java/com/victorh/poenalista/core/images/PhotoPathTest.kt`

**Interfaces:**
- Consumes: `PhotoStore` (Task 8), sessão (Task 7).
- Produces:

```kotlin
interface PhotoRemote {
    suspend fun upload(userId: String, productId: String, localPath: String): String
    suspend fun signedUrl(storagePath: String): String
}
fun photoStoragePath(userId: String, productId: String): String
```

- [ ] **Step 1: Escrever os testes que falham**

`PhotoPathTest`: `photoStoragePath("u1", "p1") == "u1/p1.jpg"`.
`FakePhotoRemote` round-trip: upload e depois `signedUrl` retorna URL contendo o caminho; upload com arquivo inexistente lança erro tratado e não registra nada.

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*PhotoPathTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

`photoStoragePath` isola a política de caminho. `SupabasePhotoRemote` usa `storage-kt`, bucket privado `fotos`, `upsert = true` e URL assinada de 3600 s. Erros são convertidos em `Result.failure` com mensagem legível.

- [ ] **Step 4: Rodar e ver passar**

Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: upload de fotos no Supabase Storage atras de interface"
```

---

### Task 10: Tela de validação ponta a ponta

**Files:**
- Create: `app/src/main/java/com/victorh/poenalista/feature/validation/ValidationScreen.kt`
- Create: `app/src/main/java/com/victorh/poenalista/feature/validation/ValidationViewModel.kt`
- Modify: `app/src/main/java/com/victorh/poenalista/ui/navigation/AppNavHost.kt` (rota inicial = validação)
- Modify: `app/src/main/java/com/victorh/poenalista/app/AppContainer.kt`
- Test: `app/src/test/java/com/victorh/poenalista/feature/validation/ValidationViewModelTest.kt`

**Interfaces:**
- Consumes: `AuthRepository`, `ProductRepository`, `PhotoStore`, `PhotoRemote`.
- Produces:

```kotlin
data class ValidationUiState(
    val email: String = "", val password: String = "",
    val name: String = "", val unit: MeasureUnit = MeasureUnit.UNIT,
    val categoryId: String? = null,
    val session: Session? = null, val loading: Boolean = false,
    val message: String? = null, val savedProductId: String? = null,
)

class ValidationViewModel(
    private val auth: AuthRepository,
    private val products: ProductRepository,
    private val categories: CategoryRepository,
    private val photoStore: PhotoStore,
    private val photoRemote: PhotoRemote,
) : ViewModel() {
    val state: StateFlow<ValidationUiState>
    fun onEmailChange(value: String)
    fun onPasswordChange(value: String)
    fun onNameChange(value: String)
    fun onUnitChange(value: MeasureUnit)
    fun onPhotoPicked(sourcePath: String)
    fun signIn()
    fun signUp()
    fun save()
}
```

- [ ] **Step 1: Escrever os testes que falham**

`ValidationViewModelTest` (fakes de todas as dependências):
- `save()` sem sessão → mensagem "Entre para salvar";
- `save()` com nome vazio → mensagem de nome obrigatório;
- ao iniciar, `categoryId` é preenchido com a categoria "Outros" (seed já executado);
- `save()` válido → produto no repositório e `savedProductId` preenchido;
- `save()` com foto e falha no upload → produto local permanece, mensagem "Foto não enviada, tentaremos depois";
- `signIn()` com `NO_CONNECTION` → mensagem "Sem conexão. Tente novamente." e estado não travado em `loading`;
- `signIn()` com sucesso → `session` preenchida.

- [ ] **Step 2: Rodar e ver falhar**

Run: `gradlew.bat testDebugUnitTest --tests "*ValidationViewModelTest"`
Expected: FAIL.

- [ ] **Step 3: Implementar**

ViewModel com `viewModelScope`, estados imutáveis e `StateFlow`. No `init`, chama `categories.seedDefaults()` e observa as categorias para fixar `categoryId` na categoria "Outros". Tela com e-mail, senha, botões entrar/criar conta, nome do produto, seletor de unidade, botão de foto (`TakePicture`) e botão salvar. Exibir mensagens em `Snackbar`. A foto é opcional: salvar sem foto é válido.

- [ ] **Step 4: Rodar e ver passar**

Run: `gradlew.bat testDebugUnitTest --tests "*ValidationViewModelTest"`
Expected: PASS.

- [ ] **Step 5: Verificação manual no aparelho**

Instale o debug APK no A54, crie uma conta real no Supabase, cadastre um produto, tire uma foto. Confirme no painel do Supabase: linha em `products` e objeto em `fotos/{userId}/{productId}.jpg`.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat: tela de validacao ponta a ponta com login, produto e foto"
```

---

### Task 11: README, runbook e fechamento da etapa

**Files:**
- Create: `README.md`
- Create: `docs/runbook-build.md`
- Modify: `docs/superpowers/specs/2026-10-23-poe-na-lista-design.md` (marcar P3 como validado)

**Interfaces:**
- Consumes: tudo.
- Produces: documentação operacional.

- [ ] **Step 1: Escrever o README**

Conteúdo mínimo: o que é o app, stack, como clonar e abrir no Android Studio, como rodar os testes, como gerar o APK de debug, limitações conhecidas (Etapa 0) e link para a spec.

- [ ] **Step 2: Escrever o runbook**

Como gerar APK assinado (quando chegar a hora), onde guardar a keystore, como instalar via `adb install`, como atualizar sem perder dados.

- [ ] **Step 3: Registrar o resultado da validação**

No documento da spec, atualizar a pendência P3 com o que foi observado: versão do `supabase-kt`, atrito encontrado, decisão de manter ou trocar por REST.

- [ ] **Step 4: Verificação final**

Run: `gradlew.bat testDebugUnitTest` e `gradlew.bat connectedDebugAndroidTest`
Expected: todos os testes passando; app instalado e funcional no A54.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "docs: README e runbook de build; valida stack na etapa 0"
```

---

## Critério de conclusão do Plano 1

- App instalado no Galaxy A54.
- Login real no Supabase funcionando, com erro claro sem internet.
- Produto salvo no Room e visível após reiniciar o app.
- Foto tirada, comprimida para ≤ 1080 px e enviada ao Storage.
- Sessão preservada ao fechar e reabrir.
- `testDebugUnitTest` e `connectedDebugAndroidTest` verdes.
- README e runbook publicados.

Quando este critério for atingido, o Plano 2 (catálogo completo e captura rápida) é escrito com as versões e os atritos reais em mãos.
