# Mobywatell-app
// screens/HomeScreen.js
import React, { useEffect, useState } from 'react';
import { View, Text, TouchableOpacity, FlatList, StyleSheet, Button, Alert } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

export default function HomeScreen({ navigation }) {
  const [documents, setDocuments] = useState([]);

  useEffect(() => {
    const loadDocuments = async () => {
      try {
        const jsonValue = await AsyncStorage.getItem('documents');
        if (jsonValue != null) {
          setDocuments(JSON.parse(jsonValue));
        } else {
          setDocuments([
            { id: '1', name: 'mDowód', details: 'Jan Kowalski\nPESEL: 90010112345\nSeria: ABC123456' },
            { id: '2', name: 'mPrawo Jazdy', details: 'Nr prawa jazdy: 123456789\nKategoria B\nWażne do: 01.01.2030' },
            { id: '3', name: 'mLegitymacja', details: 'Student: Jan Kowalski\nUczelnia: PW\nNr albumu: 123456' },
          ]);
        }
      } catch (e) {
        console.error('Błąd ładowania dokumentów', e);
      }
    };
    loadDocuments();
  }, []);

  const saveDocuments = async (docs) => {
    try {
      await AsyncStorage.setItem('documents', JSON.stringify(docs));
    } catch (e) {
      console.error('Błąd zapisu dokumentów', e);
    }
  };

  const handleLogout = async () => {
    await AsyncStorage.removeItem('user');
    navigation.replace('Login');
  };

  const handleDocumentPress = (doc) => {
    navigation.navigate('Document', { doc });
  };

  const handleDeleteDocument = (docId) => {
    Alert.alert(
      'Usuń dokument',
      'Czy na pewno chcesz usunąć ten dokument?',
      [
        { text: 'Anuluj', style: 'cancel' },
        {
          text: 'Usuń',
          style: 'destructive',
          onPress: () => {
            const filteredDocs = documents.filter(doc => doc.id !== docId);
            setDocuments(filteredDocs);
            saveDocuments(filteredDocs);
          },
        },
      ]
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Twoje dokumenty</Text>
      <FlatList
        data={documents}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={styles.card}
            onPress={() => handleDocumentPress(item)}
            onLongPress={() => handleDeleteDocument(item.id)}
          >
            <Text style={styles.cardText}>{item.name}</Text>
          </TouchableOpacity>
        )}
      />
      <View style={{ marginVertical: 20 }}>
        <Button title="Skanuj QR" onPress={() => navigation.navigate('QR', {
          addDocument: (newDoc) => {
            const updatedDocs = [...documents, newDoc];
            setDocuments(updatedDocs);
            saveDocuments(updatedDocs);
          }
        })} />
      </View>
      <Button title="Wyloguj się" onPress={handleLogout} color="red" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    marginBottom: 16,
    fontWeight: 'bold',
  },
  card: {
    padding: 16,
    backgroundColor: '#f1f1f1',
    borderRadius: 10,
    marginBottom: 12,
  },
  cardText: {
    fontSize: 18,
  },
});
