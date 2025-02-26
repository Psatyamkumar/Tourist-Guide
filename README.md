import React, { useState } from 'react';
import { Card, CardContent, Button, Input, Notification, Avatar } from '@/components/ui';
import { sendOtp, verifyOtp, sendWhatsAppNotification } from './firebase';
import { motion } from 'framer-motion';

const foods = [
  { name: 'Maggi', price: [10, 20, 30], image: '/maggi.jpg' },
  { name: 'Munchuriyan', price: [10, 20, 30], image: '/munchuriyan.gif' },
  { name: 'Pizza', price: [10, 20, 30], image: '/pizza.jpg' }
];

export default function FoodDelivery() {
  const [mobile, setMobile] = useState('');
  const [otp, setOtp] = useState('');
  const [loggedIn, setLoggedIn] = useState(false);
  const [orders, setOrders] = useState([]);

  const handleLogin = async () => {
    await sendOtp(mobile);
    alert('OTP Sent!');
  };

  const verifyLogin = async () => {
    const verified = await verifyOtp(mobile, otp);
    if (verified) {
      setLoggedIn(true);
    } else {
      alert('Invalid OTP');
    }
  };

  const placeOrder = (food, price) => {
    const newOrder = { food, price, time: new Date().toLocaleString() };
    setOrders([...orders, newOrder]);
    sendWhatsAppNotification(mobile, food, price);
    Notification.success(`${food} Order Booked Successfully!`);
  };

  return (
    <div className="bg-purple-600 min-h-screen text-white p-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl">Prajapati Ji</h1>
        {loggedIn ? <Avatar>{mobile}</Avatar> : <Input placeholder="Enter Mobile Number" onChange={e => setMobile(e.target.value)} />}
        {!loggedIn ? <Button onClick={handleLogin}>Send OTP</Button> : null}
      </div>
      {loggedIn && <Input placeholder="Enter OTP" onChange={e => setOtp(e.target.value)} />}
      {loggedIn && <Button onClick={verifyLogin}>Verify OTP</Button>}
      {loggedIn && (
        <motion.div className="grid grid-cols-3 gap-4 mt-10">
          {foods.map(food => (
            <Card key={food.name}>
              <CardContent>
                <img src={food.image} alt={food.name} className="rounded-lg mb-4" />
                <h2>{food.name}</h2>
                {food.price.map(price => (
                  <Button key={price} onClick={() => placeOrder(food.name, price)}>
                    {price} Rs
                  </Button>
                ))}
              </CardContent>
            </Card>
          ))}
        </motion.div>
      )}
    </div>
  );
}
