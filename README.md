# day-28-task
import React, { createContext, useReducer, useEffect } from 'react';
import axios from 'axios';

const CartContext = createContext();

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'SET_ITEMS':
      return {
        ...state,
        items: action.payload,
      };
    case 'INCREASE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item => item.id === action.payload ? 
          { ...item, quantity: item.quantity + 1 } : item),
      };
    case 'DECREASE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item => 
          item.id === action.payload && item.quantity > 1 ? 
          { ...item, quantity: item.quantity - 1 } : item),
      };
    default:
      return state;
  }
};

export const CartProvider = ({ children }) => {
  const initialState = {
    items: [],
  };

  const [state, dispatch] = useReducer(cartReducer, initialState);

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'https://drive.google.com/uc?export=download&id=1mBA4azCOF4ouh5iVie-Pe7F_CX2d-gYD'
      );
      dispatch({ type: 'SET_ITEMS', payload: result.data });
    };
    fetchData();
  }, []);

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
};

export default CartContext;
import React, { useContext } from 'react';
import CartContext from '../context/CartContext';

const Cart = () => {
  const { state, dispatch } = useContext(CartContext);

  const totalQuantity = state.items.reduce((total, item) => total + item.quantity, 0);
  const totalAmount = state.items.reduce((total, item) => total + item.quantity * item.price, 0);

  return (
    <div className="cart">
      <h1>Shopping Cart</h1>
      <table>
        <thead>
          <tr>
            <th>Item</th>
            <th>Price</th>
            <th>Quantity</th>
            <th>Total</th>
          </tr>
        </thead>
        <tbody>
          {state.items.map(item => (
            <tr key={item.id}>
              <td>{item.name}</td>
              <td>${item.price}</td>
              <td>
                <button onClick={() => dispatch({ type: 'DECREASE_QUANTITY', payload: item.id })}>-</button>
                {item.quantity}
                <button onClick={() => dispatch({ type: 'INCREASE_QUANTITY', payload: item.id })}>+</button>
              </td>
              <td>${(item.price * item.quantity).toFixed(2)}</td>
            </tr>
          ))}
        </tbody>
      </table>
      <div className="summary">
        <p>Total Quantity: {totalQuantity}</p>
        <p>Total Amount: ${totalAmount.toFixed(2)}</p>
      </div>
    </div>
  );
};

export default Cart;
.cart {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

.summary {
  margin-top: 20px;
  font-size: 18px;
}

button {
  padding: 5px 10px;
  margin: 0 5px;
  cursor: pointer;
}
import React from 'react';
import { CartProvider } from './context/CartContext';
import Cart from './components/Cart';
import './styles.css';

function App() {
  return (
    <CartProvider>
      <Cart />
    </CartProvider>
  );
}

export default App;
