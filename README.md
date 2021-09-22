# Clustering
# Alumno Matias Nahuel Pereyra
# Materia "Text Mining" FAMAF-UNC 2021

Agrupamiento de palabras según su relación sintáctica y semántica utilizando el argoritmo de k-means.

El corpus ultilizado es del diario La Voz del Interior.

## Procedimiento
### Procesamiento del corpus
1. Dividimos el corpus en oraciones y en palabras.
2. Eliminamos las oraciones con menos de 5 palabras.
3. Lematizamos las palabras y eliminamos las palabras que no son alpha.
4. Creamos un diccionario de palabras.
    * Agregamos las palabras que no sean números, o desconocidas. Cada palabra contiene un diccionario.
    * Agregamos el diccionario de cada palabra.
    * Eliminamos las palabras poco frecuentes en el corpus, de los diccionarios de las palabras.
    * Eliminamos las palabras poco frecuentes como contexto, de los diccionarios de las palabras.
5. Vectorizamos las palabras.
6. Normalizamos la matriz.

### Clustering
1. Elegimos un número de clusters.
2. Fijamos centroides aleatorios.
3. Utilizamos el algoritmo de k-means usando la distancia coseno para crear los clusters.
4. Testeamos diferentes valores de k.

## Explicacion paso por paso
Utilizamos Spacy

 nlp = spacy.load("es_core_news_sm")

Borramos las oraciones con menos de 5 palabras:

      sents = [sent for sent in doc.sents if len(sent) > 10]

Creamos una lista con las palabras lematizadas, evitando aquellas que sean puntuaciones o números y contamos las ocurrencias de cada palabra.

        words = []
        words_lemma = []
        for sent in sents:
            for word in sent:
                if word.is_alpha:
                    words.append(word)
                    words_lemma.append(word.lemma_)
                    
        counts = Counter(words_lemma)
      

Generamos un diccionario con una ventana de 2 palabras, limitandonos a palabras que aparezcan mas de 60 veces y a 30 palabras del contexto.

    def featureContext(features ,context):
    for c_word in context:       
        c_lemma = c_word.lemma_ 
        key = c_lemma 
        if c_word.is_alpha and counts[c_lemma] > threshold_c: 
            if c_word.like_num: 
                key = "NUM__" 
            if not key in features: 
                features[key] = 0 
            features[key] += 1
    
Diccionario:

    dicc = {} 
    threshold_w = 60 
    threshold_c = 30 
    
    for word in words: 
        w = word.lemma_ 
        if word.like_num or counts[w] < threshold_w: 
            continue 
        if not w in dicc: 
            features = {} 
        else: 
            features = dicc[w] 
    
        pos = "POS__" + word.pos_ 
        if not pos in features: 
            features[pos] = 0 
        features[pos] += 1 
    
        dep = "DEP__" + word.dep_ 
        if not dep in features: 
            features[dep] = 0 
        features[dep] += 1 

        if not word.i == 0: 
        context_left = doc[word.i - 2: word.i] if word.i > 2 else doc[word.i - 1: word.i] 
        featureContext(features, context_left)

        if not word.i == len(doc) - 1:
        context_right = doc[word.i + 1 : word.i + 3] if word.i < len(doc) - 2 else doc[word.i + 1 : word.i + 2]
        featureContext(features, context_left)

        dicc[w] = features

Separamos las palabras y sus features

      features = []
      key_words = {}
      wid = 0
      for d in dicc:
          if len(d) > 0:
              key_words[d] = wid
              wid += 1
              features.append(dicc[d])
              
Vectorizamos las palabras con Sklearn

      from sklearn.feature_extraction import DictVectorizer
      v = DictVectorizer(sparse=False)
      matrix = v.fit_transform(features)
      
Normalizamos la matriz

      matrix_normed = matrix / matrix.max(axis=0)
      
Reducimos la dimensionalidad quitando las columnas que tengan poca varianza.

      variances = np.square(matrix_normed).mean(axis=0) - np.square(matrix_normed.mean(axis=0))
      threshold_v = 0.001
      red_matrix = np.delete(matrix_normed, np.where(variances < threshold_v), axis=1)
      
Creamos los clusters: 

      clusterer = kmeans.KMeansClusterer(k, cosine_distance, avoid_empty_clusters=True)
      clusters = clusterer.cluster(red_matrix, True)

## Resultados
## k = 50

### Stopwords
['él', 'que', 'a', 'el', 'ser', 'y', 'sin', 'otro', 'en', 'de', 'del', 'sino', 'tanto', 'uno', 'cuando', 'pero', 'como', 'para', 'con', 'todo', 'ese', 'al', 'través', 'también', 'por', 'contra', 'cada', 'o', 'aunque', 'ayer', 'este', 'desde', 'entre', 'porque', 'alguno', 'su', 'recién', 'mucho', 'donde', 'durante', 'sobre', 'luego', 'algo', 'varios', 'e', 'casi', 'allí', 'según', 'cerca', 'cualquiera', 'dentro', 'En', 'hacia', 'frente']

### Dias de semana / tiempo
['año', 'momento', 'manera', 'día', 'toma', 'miércoles', 'debate', 'martes', 'mes', 'noche', 'discusión', 'caso', 'diciembre', 'década', 'declaración', 'viernes', 'hora', 'reunión', 'agosto', 'informe', 'diario', 'jueves', 'período', 'semana', 'escenario', 'denuncia', 'julio', 'mensaje', 'elección', 'jornada', 'noviembre', 'comunicado', 'domingo']

### Nombres
['tú', 'Daniel', 'Carlos', 'denunciar', 'tránsito', 'municipal', 'Néstor', 'Juan', 'Eduardo', 'Ricardo', 'Marcelo', 'solicitar', 'entregar']

### Funcionarios públicos
['colegio', 'acción', 'construcción', 'De', 'funcionario', 'decisión', 'obtener', 'anuncio', 'paso', 'secretario', 'fallo', 'artículo', 'producir', 'alguien', 'resolución', 'Mario', 'concurso']

## k = 100

### Stopwords
['él', 'que', 'a', 'el', 'ser', 'y', 'no', 'sin', 'otro', 'estar', 'en', 'de', 'del', 'sino', 'yo', 'uno', 'pero', 'ya', 'más', 'como', 'para', 'con', 'ese', 'al', 'después', 'también', 'por', 'si', 'cada', 'ahora', 'poder', 'nunca', 'este', 'desde', 'entre', 'alguno', 'su', 'mucho', 'deber', 'respecto', 'sobre', 'hasta', 'antes', 'apenas', 'atrás', 'además', 'según', 'mientras', 'entonces', 'aún', 'nadie']

### Dias de semana / tiempo
['día', 'miércoles', 'martes', 'lunes', 'jueves', 'período', 'semana', 'anuncio', 'paso', 'elección', 'noviembre', 'domingo']

### Nombres
['Daniel', 'Cristina', 'Juan', 'Schiaretti', 'Eduardo', 'Marcelo', 'Jorge', 'Mario']

### Funcionarios públicos
['adolescente', 'jefe', 'interior', 'presidente', 'funcionario', 'dirigente', 'legislador', 'titular', 'Néstor', 'gobernador', 'senador', 'diputado', 'secretario']